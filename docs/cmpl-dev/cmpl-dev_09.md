

## 第八章：8 使用 JEST 框架进行测试



![](img/Drop-image.jpg)

每当你修改代码时，你都面临着可能在应用程序的另一部分引发无法预见的副作用的风险。因此，保证代码库的完整性和稳定性可能具有挑战性。为了做到这一点，开发者遵循两种主要策略。

第一种策略是一种架构模式，我们将代码拆分为小的、自包含的 React 组件。这些组件本质上不会相互干扰。因此，修改其中一个不应该导致任何副作用。第二种策略是进行自动化单元测试，本章将介绍如何使用 Jest 框架进行测试。

在接下来的章节中，我们将讨论自动化单元测试的基本要素以及使用它的好处。你将学习如何在 Jest 中编写测试套件，并利用其报告来改进代码。你还将通过使用代码替身来处理依赖关系。最后，你将探索可能想要在应用程序中运行的其他类型的测试。

### 测试驱动开发和单元测试

开发者有时使用*测试驱动开发（TDD）*的技术，在实际编写待测试的代码之前先编写自动化测试。他们首先创建一个测试来评估最小的代码单元是否按预期工作。这样的测试被称为*单元测试*。接着，他们编写通过测试所需的最少量代码。

这种方法有明显的好处。首先，它让你通过明确地定义代码的功能和边界情况，集中精力于应用程序的需求。因此，你能清楚地了解其期望的行为，并且能更早地发现不明确或缺失的规范。当你在完成功能后编写测试时，它们可能反映的是你实现的行为，而不是你所需要的行为。

其次，限制自己只编写必要的代码可以防止你的函数变得过于复杂，并将你的应用程序拆分成小而易于理解的部分。可测试的代码是可维护的代码。此外，这种技术确保你的测试覆盖了应用程序代码的很大一部分，这个指标称为*代码覆盖率*，并且通过在开发过程中频繁运行测试，你会立刻识别出新代码行引入的错误。

根据情况，单元测试所针对的*单元*可以是一个模块、一个函数或一行代码。测试的目的是验证每个单元在独立情况下是否正常工作。每个测试函数中的单行代码就是测试的*步骤*，整个测试函数被称为一个测试*用例*。测试*套件*将多个测试用例聚合成逻辑块。要被认为是可重复的，测试必须在每次运行时返回相同的结果。正如我们在本章中将要探讨的那样，这意味着我们必须在一个控制的环境中运行测试，并使用定义好的数据集。

Facebook 与 React 一起开发了 Jest 测试框架，但我们可以在任何 Node.js 项目中使用它。它有一套定义好的语法来设置和编写测试。其*测试运行器*执行这些测试，自动替换代码中的任何依赖项，并生成测试覆盖率报告。额外的 npm 模块提供了测试 DOM 或 React 组件的自定义代码，当然，也可以添加 TypeScript 类型。

### 使用 Jest

要在项目中使用 Jest，我们必须安装所需的包，创建一个用于存放所有测试文件的目录，并添加一个 npm 脚本来运行测试。在 Next.js 应用程序的根目录中执行以下命令，安装框架以及来自 DefinitelyTyped 的类型定义作为开发依赖：

```
$ **npm install --save-dev jest @types/jest**
```

然后，创建一个目录来保存你的测试。Jest 默认使用 *__tests__* 文件夹，因此在根目录下创建一个。接下来，为了将 npm 脚本 *test* 添加到你的项目中，打开 *package.json* 文件，并修改 scripts 对象，使其与 清单 8-1 中的内容匹配。

```
 "scripts": {
      "dev": "next dev",
      "build": "next build",
      "start": "next start",
      "lint": "next lint"**,**
  **"test": "jest"**
  }, 
```

清单 8-1：带有新文本命令的 package.json 文件

现在我们可以使用 npm test 命令运行测试。通常，构建服务器在构建过程中默认会执行此命令。最后，为了在 Jest 中启用 TypeScript 支持，请添加 ts-jest 转译器：

```
$ **npm install --save-dev ts-jest**
```

还要创建一个 *jest.config* 文件，通过运行 npx ts-jest config:init 来添加 TypeScript。

### 创建一个示例模块以进行测试

让我们编写一些示例代码，帮助我们理解单元测试和 TDD。假设我们想在应用程序中创建一个新模块，*./helpers/sum.ts*。它应该导出一个名为 sum 的函数，返回其参数的总和。为了遵循 TDD 模式，我们将首先为这个模块创建测试用例。

首先，我们需要创建一个函数来运行我们的测试。在默认测试目录中创建一个名为 *sum.test.ts* 的文件，并添加 清单 8-2 中的代码。

```
import {sum} from "../helpers/sum";

describe("the sum function", () => {

}); 
```

清单 8-2：空测试套件

我们导入稍后将编写的 sum 函数，并使用 Jest 的 describe 函数创建一个空的测试套件。当我们使用 npm test 运行（不存在的）测试时，Jest 应该会抱怨在 *helpers* 目录中没有名为 *sum.ts* 的文件。现在在你的项目根目录中创建这个文件和文件夹。在文件中编写 Listing 8-3 所示的 sum 函数。

```
const sum = () => {};
export {sum}; 
```

Listing 8-3：sum 函数的基础骨架

现在再次使用 npm test 运行测试。由于代码只是导出了一个返回空值的占位符 sum 函数，Jest 测试运行器再次抱怨。这次，它通知我们测试套件需要包含至少一个测试。

让我们来看看一个测试用例的结构，并在这个过程中向 *sum.test.ts* 文件添加一些测试用例。

### 测试用例的结构

单元测试有两种类型：基于状态的和基于交互的。*基于交互的*测试用例验证被评估的代码是否调用了特定的函数，而*基于状态的*测试用例检查代码的返回值或结果状态。两种类型都遵循相同的三个步骤：安排、执行和断言。

#### 安排

为了编写独立且可复现的测试，我们需要首先*安排*我们的环境，定义前提条件，例如测试数据。如果我们仅在一个特定的测试用例中需要这些前提条件，我们会在该用例开始时定义它们。否则，我们通过使用 beforeEach 钩子（该钩子在每个测试用例之前执行）或 beforeAll 钩子（该钩子在所有测试运行之前执行）将它们为所有测试在测试套件中设置为全局。

举例来说，如果我们有某种原因需要在每个测试用例中使用相同的全局数据集，并且知道我们的测试步骤会修改数据集，那么我们需要在每次测试前重新创建数据集。beforeEach 钩子是执行此操作的最佳位置。另一方面，如果测试用例只是消费数据，那么我们只需要定义一次数据集，因此可以使用 beforeAll 钩子。

让我们定义两个测试用例，并为每个用例创建输入值。我们的输入参数将针对每个测试用例进行特定定义，因此我们将在测试用例内部声明它们，而不是使用 `beforeEach` 或 `beforeAll` 钩子。使用 Listing 8-4 中的代码更新 *sum.test.ts* 文件。

```
import {sum} from "../helpers/sum";

describe("the sum function", () => {
    **test("two plus two is four", () => {**
 **let first =** **2;**
 **let second = 2;**
 **let expectation = 4;**
    });

    **test("minus eight plus four is minus four", () => {**
 **let first = -8;**
 **let second =** **4;**
 **let expectation = -4;**
    });
}); 
```

Listing 8-4：包含安排步骤的测试套件

`describe` 函数创建我们的测试套件，其中包含两次调用 `test` 函数，每次调用都代表一个测试用例。对于这两个用例，第一个参数是我们在测试运行器报告中看到的描述信息。

我们的每个测试都评估 `sum` 函数的结果。第一个测试检查加法功能，验证 2 加 2 是否返回 4。第二个测试确认该函数也能正确返回负值。它将 4 加到 −8，并期望返回 −4。

你可能还想检查 `sum` 函数的返回类型。通常，我们会检查返回类型，但由于我们正在使用 TypeScript，因此不需要这个额外的测试用例。相反，我们可以在函数签名中定义返回类型，TSC 会为我们验证它。

#### 操作

一旦测试运行器执行某个用例，测试步骤就会通过使用特定测试用例的数据来调用待测试的代码，*代表*我们执行相应的操作。每个测试用例应测试系统的一个功能或变体。这个步骤是调用执行函数的代码行。Listing 8-5 将其添加到 *sum.test.ts* 的测试用例中。

```
import {sum} from "../helpers/sum";

describe("the sum function", () => {

    test("two plus two is four", () => {
        let first = 2;
        let second = 2;
        let expectation = 4;
 **let result = sum(first, second);**
    });

    test("minus eight plus four is minus four", () => {
        let first = -8;
        let second = 4;
        let expectation = -4;
 **let result = sum(first, second);**
    });

}); 
```

Listing 8-5：包含操作步骤的测试套件

我们的新代码行调用 `sum` 函数，并将我们定义的参数值传递给它。我们将返回的值存储在 `result` 变量中。在编辑器中，TSC 应该会抛出类似 `Expected 0 arguments, but got 2` 的错误。这是正常的，因为 `sum` 函数只是一个空占位符，尚未期望任何参数。

#### 断言

我们测试用例的最后一步是*断言*，即代码满足我们定义的预期。我们通过两个部分创建这个断言：Jest 的expect函数，结合 Jest 的*assert*库中的*matcher*函数来定义我们测试的条件。根据单元测试的类别，这个条件可以是特定的返回值、状态变化或调用另一个函数。常见的匹配器检查值是否为数字、字符串等。我们还可以使用它们来断言一个函数返回 true 或 false。

Jest 的*assert*库为我们提供了一组内置的基本匹配器，我们可以从 npm 仓库中添加额外的匹配器。最常见的断言包之一是*testing-library/dom*，用于查询 DOM 中特定的节点并断言其特性。例如，我们可以检查类名或属性，或者与原生 DOM 事件一起使用。另一个常见的断言包是*testing-library/react*，它为 React 提供了实用工具，并让我们在断言中访问render函数和 React hooks。

因为每个测试用例评估一个代码单元中的一个条件，我们将每个测试限制为一个断言。这样，一旦测试运行成功或失败，测试报告生成时，我们可以轻松找出哪个测试假设失败了。清单 8-6 为每个测试用例添加了一个断言。将它粘贴到*sum.test.ts*文件中。

```
import {sum} from "../helpers/sum";

describe("the sum function", () => {

    test("two plus two is four", () => {
        let first = 2;
        let second = 2;
        let expectation = 4;
        let result = sum(first, second);
 **expect(result).toBe(expectation);**
    });

    test("minus eight plus four is minus four", () => {
        let first = -8;
        let second = 4;
        let expectation = -4;
        let result = sum(first, second);
 **expect(result).toBe(expectation);**
    });

}); 
```

清单 8-6：包含断言步骤的测试套件

这些行使用expect断言函数，并与toBe匹配器一起使用，以将预期结果与我们的期望进行比较。我们的测试用例现在已经完成。每个测试用例都遵循*arrange, act, assert*模式，并验证一个条件。附录 C 列出了其他匹配器。

### 使用 TDD

我们的测试用例仍未执行，如果你运行npm test，测试运行器应该会立即失败。TSC 检查代码，并且由于缺少对sum函数的参数声明，它会抛出错误：

```
FAIL  __tests__/sum.test.ts
  • Test suite failed to run
`--snip--`
Test Suites: 2 failed, 2 total
Tests:       0 total
Snapshots:   0 total 
```

是时候实现这个sum函数了。按照 TDD 的原则，我们将逐步向代码中添加功能，并在每次添加后运行测试套件，直到所有测试通过。首先，我们将添加那些缺失的参数。将*sum.ts*中的代码替换为清单 8-7 的内容。

```
const sum = (a: number, b: number) => {};

export {sum}; 
```

清单 8-7：带有附加参数的sum函数

我们添加了参数并将其类型指定为数字。现在我们重新运行测试用例，正如预期的那样，它们失败了。控制台输出告诉我们 sum 函数没有返回预期的结果。这不应令我们感到惊讶，因为我们的 sum 函数根本没有返回任何值：

```
FAIL  __tests__/sum.test.ts (5.151 s)
  the sum function
    × two plus two is four (6 ms)
    × minus eight plus four is minus four (1 ms)

  • the sum function › two plus two is four
    Expected: 4
    Received: undefined

  • the sum function › minus eight plus four is minus four
    Expected: -4
    Received: undefined

Test Suites: 1 failed, 1 total
Tests:       2 failed, 2 total
Snapshots:   0 total
Time:        5.328 s, estimated 11 s 
```

列表 8-8 中的代码将此功能添加到 *sum.ts* 文件中。我们将函数的返回类型指定为数字，并添加了两个参数。

```
const sum = (a: number, b: number): number => a + b;

export {sum}; 
```

列表 8-8：完整的 sum 函数

如果我们重新运行 npm test，Jest 应该报告所有测试用例都通过了：

```
PASS  __tests__/sum.test.ts (8.045 s)
  the sum function
    ✓ two plus two is four (2 ms)
    ✓ minus eight plus four is minus four (2 ms)

Test Suites: 1 passed, 1 total
Tests:       2 passed, 2 total
Snapshots:   0 total
Time:        8.291 s 
```

如你所见，一切正常。

#### 重构代码

单元测试在我们需要重构代码时特别有用。例如，我们可以重写 sum 函数，使其接受一个数字数组，而不是两个参数。该函数应返回数组中所有项的和。

我们首先将现有的测试用例重写为更简洁的形式，然后扩展测试套件以验证新行为。将 *sum.test.file* 中的代码替换为 列表 8-9。

```
import {sum} from "../helpers/sum";

describe("the sum function", () => {

    test("two plus two is four", () => {
 **expect(sum([2, 2])).toBe(4);**
    });

    test("**minus eight** plus **four** is **minus four**", () => {
        expect(sum([**-8**, **4**])).toBe(**-4**);
    });

    test("**two** plus **two** plus **minus four** is **zero**", () => {
        expect(sum([**2**, **2**, **-4**])).toBe(0);
    });

}); 
```

列表 8-9：重构后的 sum 函数的测试套件

请注意，我们将测试用例重写为更简洁的形式。虽然将 arrange、act 和 assert 语句拆分到多行可能更易于阅读，但对于像 列表 8-9 中的简单测试用例，我们通常会将其写成一行。我们已将其功能进行了更改，以适应新需求。我们的 sum 函数不再接受两个值，而是接受一个包含数字的数组。再次提醒，TSC 会立即通知我们测试套件中的 sum 函数与实际实现之间的参数不匹配。

一旦编写了测试用例，我们就可以重写代码了。列表 8-10 展示了 *helpers/sum.ts* 文件的代码。在这里，sum 函数现在接受一个数字数组作为参数，并返回一个数字。

```
const sum = (data: number[]): number => {
    return data[0] + data[1];
};

export {sum}; 
```

列表 8-10：在 helpers/sum.ts 文件中重写的 sum 函数

我们将参数更改为一个数字数组。这修复了由 列表 8-9 中的测试套件引起的 TypeScript 错误。但因为我们遵循 TDD 并且每次只做一个功能性更改，我们保持了函数原有的行为，即添加两个值。正如预期的那样，当我们使用 npm test 运行自动化测试时，测试用例中的一个会失败：

```
FAIL  __tests__/sum.test.ts (7.804 s)
  the sum function
    ✓ two plus two is four (7 ms)
    ✓ minus eight plus four is minus four (1 ms)
    ✕ two plus two plus minus four is zero (9 ms)

  • the sum function › two plus two plus minus four is zero
    Expected: 0
    Received: 4

Test Suites: 1 failed, 1 total
Tests:       1 failed, 2 passed, 3 total
Snapshots:   0 total
Time:        8.057 s, estimated 9 s 
```

测试新需求的第三个测试用例失败了。我们不仅预期到这个结果，而且希望测试失败；这样我们就能确认测试本身有效。如果在我们实现相应功能之前测试就通过了，那么测试用例就是错误的。

以失败的测试作为基准，现在是时候重构代码以适应新的需求了。将 列表 8-11 中的代码粘贴到 *sum.ts* 文件中。在这里，我们重构了 sum 函数，使其返回所有数组值的和。

```
const sum = (data: number[]): number => {
    return data.reduce((a, b) => a + b);
};

export {sum}; 
```

列表 8-11：修正后的 sum 函数，使用 array.reduce

尽管我们可以使用 for 循环遍历数组，但我们使用现代 JavaScript 的 array.reduce 函数。这个原生数组函数会对每个数组元素运行一个回调函数。回调函数接收上一次迭代的返回值和当前数组项作为参数：这正是我们计算和所需的。

运行测试套件中的所有测试用例，验证它们是否按预期工作：

```
PASS  __tests__/sum.test.ts (7.422 s)
  the sum function
    ✓ two plus two is four (2 ms)
    ✓ minus eight plus four is minus four
    ✓ two plus two plus minus four is zero

Test Suites: 1 passed, 1 total
Tests:       3 passed, 3 total
Snapshots:   0 total
Time:        7.613 s 
```

测试运行器应显示代码通过了所有测试。

#### 评估测试覆盖率

为了准确测量我们的测试套件覆盖了哪些代码行，Jest 会生成测试覆盖率报告。我们的测试评估的代码比例越高，测试就越全面，关于应用程序质量和可维护性的信心也越强。作为一般经验法则，您应该争取达到 90% 或以上的代码覆盖率，并且在最关键的部分有较高的覆盖率。当然，测试用例应该通过测试代码的功能来增加价值；单纯为了增加测试覆盖率而添加测试并非我们的目标。但一旦彻底测试了代码库，您就可以重构现有功能并实现新功能，而不必担心引入回归性错误。高代码覆盖率验证了更改没有隐藏的副作用。

修改 *package.json* 文件中的 npm test 脚本，添加 --coverage 标志，如 列表 8-12 所示。

```
 "scripts": {
      "dev": "next dev",
      "build": "next build",
      "start": "next start",
      "lint": "next lint"**,**
  **"test": "jest --coverage"**
  }, 
```

列表 8-12：在 package.json 文件中启用 Jest 的测试覆盖率功能

如果我们重新运行测试套件，Jest 应该会显示我们的单元测试覆盖了代码的百分比。它会生成一个代码覆盖率报告并将其存储在*coverage*文件夹中。请将你的输出与以下内容进行比较：

```
PASS  __tests__/sum.test.ts (7.324 s)
  the sum function
    ✓ two plus two is four (2 ms)
    ✓ minus eight plus four is minus four
    ✓ two plus two plus minus four is zero (1 ms)
----------|---------|----------|---------|---------|-------------------
File      | % Stmts | % Branch | % Funcs | % Lines | Uncovered Line #s
----------|---------|----------|---------|---------|-------------------
All files |     100 |      100 |     100 |     100 |
  sum.ts  |     100 |      100 |     100 |     100 |
----------|---------|----------|---------|---------|-------------------
Test Suites: 1 passed, 1 total
Tests:       3 passed, 3 total
Snapshots:   0 total
Time:        7.687 s, estimated 8 s 
```

报告显示了按语句、分支、函数和行分解的覆盖率。我们看到我们简单的 sum 函数在所有类别中的代码覆盖率为 100%。因此，我们知道我们没有留下任何未测试的代码，并且可以信任测试用例反映了函数的质量。

### 用伪造、存根和模拟替代依赖关系

我们提到过，我们的测试应该在独立的环境中运行，而不依赖于外部代码。你可能会想，如何处理导入的模块？毕竟，一旦你导入代码，就会为被评估的单元添加一个依赖关系。这些第三方模块可能无法按预期工作，我们不希望我们的代码依赖于假设它们都能正确运行。因此，你应该为每个导入的模块提供一组测试用例来验证其功能。它们也是需要测试的单元。

单独地，我们需要用*测试替身*替换我们其他代码单元中的模块，而不是导入它们，测试替身返回一组针对测试的静态数据。测试替身替代了一个对象或函数，有效地消除了依赖关系。由于它们返回的是已定义的数据集，因此它们的响应是已知的且可预测的。你可以把它们比作电影中的替身演员。

除了替代对象或函数外，测试替身还有第二个重要目的：它们记录它们的调用并允许我们对其进行监控。因此，我们可以用它们来测试测试替身是否被调用过，调用了多少次，以及接收到哪些参数。测试替身有三种主要类型：伪造、存根和模拟。然而，你有时会听到*模拟*一词用于指代这三者。

#### 创建一个带有依赖关系的模块

为了在我们的 sum 函数中练习使用测试替身，我们将创建一个新函数，计算斐波那契数列中的指定数量值。*斐波那契数列*是一个模式，其中每个后续数字是前两个数字的和，这是 sum 模块的一个简单用例。

所有开发者都必须确定他们的测试用例需要多么精细。斐波那契数列就是一个很好的例子，因为尝试测试提交给函数的每一个可能的数字是没有意义的，因为数列是无限的。相反，我们希望验证函数是否正确处理边界情况，并且其底层功能是否正常工作。例如，我们将检查它如何处理长度为 0 的输入；在这种情况下，函数应该返回一个空字符串。然后，我们将测试它如何计算任意长度大于 3 的斐波那契数列。在 *__tests__* 文件夹中创建 *fibonacci.test.ts* 测试套件，然后将示例 8-13 中的代码添加进去。

```
import {fibonacci} from "../helpers/fibonacci";

describe("the fibonacci sequence", () => {

    test("with a length of 0 is ", () => {
        expect(fibonacci(0)).toBe(" ");
    });

    test("with a length of 5 is '0, 1, 1, 2, 3' ", () => {
        expect(fibonacci(5)).toBe("0, 1, 1, 2, 3");
    });

}); 
```

示例 8-13：fibonacci 函数的测试套件

我们定义了两个测试用例：一个检查长度为 0 的输入，另一个计算一个包含五个数字的斐波那契数列。两个测试都遵循我们之前使用的紧凑版 *arrange, act, assert* 模式。

创建完测试用例后，我们可以继续编写斐波那契函数的代码。在 *helpers* 文件夹中创建 *fibonacci.ts* 文件，放在 *sum.ts* 文件旁边，然后将示例 8-14 中的代码添加进去。

```
import {sum} from "./sum";

const fibonacci = (length: number): string => {
    const sequence: number[] = [];
    for (let i = 0; i < length; i++) {
        if (i < 2) {
            sequence.push(sum([0, i]));
        } else {
            sequence.push(sum([sequence[i - 1], sequence[i - 2]]));
        }
    }
    return sequence.join(", ");
};

export {fibonacci}; 
```

示例 8-14：fibonacci 函数

我们从本章早些时候创建的模块中导入了 sum 函数。它现在是一个依赖项，稍后我们需要将其替换为测试双重对象。接着，我们实现了 fibonacci 函数，该函数接受要计算的数列长度并返回一个字符串。我们将当前的数列存储在一个数组中，以便能够简单地访问计算下一个数值所需的两个前一个值。请注意，数列中的第一个数字始终是 0，第二个是 1。最后，我们返回一个包含所请求数量值的字符串。如果你保存这段代码并重新运行测试套件，*sum.test.js* 和 *fibonacci.test.ts* 都应该成功通过。

#### 创建一个 Doubles 文件夹

因为我们在斐波那契模块中导入了 sum 函数，所以我们的代码有一个外部依赖。这对测试来说是个问题：如果 sum 函数坏了，斐波那契数列的测试也会失败，即使斐波那契实现的逻辑是正确的。

为了将测试与依赖项解耦，我们将用一个测试替代品替换*fibonacci.ts*文件中的sum函数。Jest 可以在测试运行期间替换任何模块，只要该模块在与测试文件相邻的*__mocks__*子目录中有一个同名的文件。在测试文件旁的*helpers*文件夹中创建这样的文件夹，并在其中放置一个*sum.ts*文件。现在先将文件留空。

为了启用测试替代品，我们调用jest.mock函数，并传递测试文件中保存的原始模块的路径。在 Listing 8-15 中，我们将此调用添加到*fibonacci.test.ts*文件中。

```
import {fibonacci} from "../helpers/fibonacci";

**jest.mock("../helpers/sum");**

describe("the fibonacci sequence", () => {
    test("with a length of 0 is ", () => {
        expect(fibonacci(0)).toBe(" ");
    });
    test("with a length of 5 is '0, 1, 1, 2, 3' ", () => {
        expect(fibonacci(5)).toBe("0, 1, 1, 2, 3");
    });

}); 
```

Listing 8-15：带有测试替代品的fibonacci函数的测试套件

这行代码替换了sum模块为测试替代品。现在我们来创建三种基本类型的测试替代品，将它们的代码添加到*__mocks__*文件夹中的文件中。

#### 使用存根

存根仅仅是返回一些预定义数据的对象。这使得它们非常容易实现，但使用上有所限制；通常，返回相同的数据不足以模拟依赖项的原始行为。Listing 8-16 展示了sum函数测试替代品的存根实现。将代码粘贴到*__mocks__*文件夹中的*sum.ts*文件中。

```
const sum = (data: number[]): number => 999;

export {sum}; 
```

Listing 8-16：sum函数的存根

存根函数具有与原始函数相同的签名。它接受相同的参数——一个数字数组，并返回一个字符串。然而，与原始函数不同的是，这个测试替代品总是返回相同的数字 999，无论它接收到的数据是什么。

要成功运行带有此存根函数的测试套件，我们需要调整对代码行为的预期。它不会返回斐波那契数列中的五个数字，而是会生成字符串999, 999, 999, 999, 999。如果我们看到这样的字符串，我们就知道sum函数被调用了五次。试验这个存根，修改测试套件的预期，以匹配它。然后将匹配器恢复到 Listing 8-15 中显示的状态，这样你就可以在接下来的测试中使用它们。

#### 使用伪造

伪造是最复杂的测试替代品类型。它们是原始功能的工作实现，但与真实实现不同，伪造只提供单元测试所需的功能。它们的实现被简化，通常不会处理边缘情况。

sum 的伪造通过手动添加数组中的第一个和第二个项，而不是使用 array.reduce。这种简化的实现剥夺了 sum 函数对两个以上数据点求和的能力，但对于斐波那契序列来说是足够的。减少的复杂性使其更易于理解，并且不容易出错。将 *__mocks__* 文件夹中的 *sum.ts* 文件内容替换为 Listing 8-17 中的代码。

```
const sum = (data: number[]): number => {
    return data[0] + data[1];
}
export {sum}; 
```

Listing 8-17：sum 函数的伪造

我们的伪造使用一个简单的数学加法运算符（+）来添加 data 参数中的第一个和第二个项。它的主要好处是返回的结果类似于实际实现的结果。我们现在可以运行测试套件，它们应该会成功通过，无需调整期望值，返回斐波那契序列。

#### 使用模拟

模拟介于存根和伪造之间。虽然比伪造简单，但它们返回比存根更真实的数据。虽然它们没有模拟依赖项的真实行为，但它们能够响应收到的数据。

例如，我们的简单模拟实现的 sum 函数将从一个硬编码的哈希映射中返回结果。将 Listing 8-18 中的代码替换到 *__mocks__/sum.ts* 文件中，该代码检查请求并允许斐波那契计算器使用原始的测试套件。

```
type resultMap = {
    [key: string]: number;
}

const results : resultMap= {
    "0 + 0": 0,
    "0 + 1": 1,
    "1 + 0": 1,
    "1 + 1": 2,
    "2 + 1": 3
};

const sum = (data: number[]): number => {
    return results[data.join("+")];
}

export {sum}; 
```

Listing 8-18：sum 函数的模拟

我们创建了一个类型，称为 resultMap，它使用字符串作为键，数字作为值。然后，我们使用新创建的类型来表示一个哈希映射，存储我们期望的响应。接下来，我们定义一个与原始实现具有相同接口的模拟函数。在模拟函数中，我们根据收到的参数计算出要在哈希映射中使用的键。这使我们能够返回正确的数据集，并生成一个实际的斐波那契序列。使用模拟相对于 sum 的主要好处是我们可以控制它的结果，因为它是从已知数据集返回的值。

方便的是，Jest 为我们提供了帮助工具来使用测试替代品。jest.mock函数将导入的模块替换为模拟对象。jest.fn API 创建一个基本的模拟，可以返回任何预定义的数据，而jest.spyOn让我们在不修改函数的情况下记录对其的调用。我们将在第 146 页的练习 8 中使用这些工具。

在典型的开发人员环境中，你不会过多关注存根（stubs）、假对象（fakes）和模拟（mocks）之间的细微差别，通常会把*模拟*作为测试替代品的统称。不要花太多时间在过度设计模拟上；它们只是帮助你测试代码的工具。

### 其他类型的测试

本章至此涵盖的测试是你作为全栈开发人员最常遇到的测试类型。本节简要解释了其他类型的测试以及何时使用它们。这些测试并不是要替代单元测试；而是通过覆盖实现中其他无法测试的特定方面来补充单元测试。例如，由于单元测试在隔离环境中运行，它们无法评估模块之间的交互。理论上，如果每个函数和模块都通过了测试，那么整个程序应该按预期工作。实际上，你经常会遇到由于模块文档错误导致的问题。通常，文档会声称某个 API 返回某种特定类型，但实际实现返回的是不同的类型。

#### 功能测试

虽然单元测试从开发人员的角度检查功能的实现，*功能测试*从用户的角度验证代码是否按用户预期的方式工作。换句话说，这些测试检查给定的输入是否会产生预期的输出。大多数功能测试属于*黑箱*测试的一种，它忽略模块的内部代码、副作用和中间结果，只测试接口。功能测试不会生成代码覆盖率报告。通常，质量保证经理会在系统测试阶段编写和使用功能测试。相比之下，开发人员在开发过程中编写和使用单元测试。

#### 集成测试

你已经了解了单元测试的目标是检查代码中最小的独立部分。*集成测试*则完全相反。它验证整个子系统的行为，无论是代码的层次结构，比如应用的数据存储机制，还是由多个模块组成的特定功能。集成测试检查子系统在当前环境中的集成情况。因此，它们永远不会在隔离环境中运行，通常也不使用测试替代品。

集成测试有助于发现三种类型的问题。第一类是与*模块间通信*相关的问题，即模块之间的通信。常见问题包括内部 API 集成故障和未检测到的副作用，例如某个函数没有在写入新数据到文件系统之前删除旧文件。第二类是与*环境*相关的问题，指的是代码运行的硬件和软件设置。不同的软件版本或硬件配置可能会给你的代码带来重大问题。全栈开发人员最常遇到的问题是 Node.js 版本的差异以及模块中过时的依赖项。

第三类是与*网关通信*相关的问题，指的是测试与第三方 API 网关的任何 API 通信。与外部 API 的任何通信都应该通过集成测试进行测试。这是唯一一个可能使用测试替代品的集成测试实例，比如使用外部 API 的虚拟版本，以模拟特定的 API 行为，如超时或成功请求。与功能测试一样，质量保证经理通常编写并使用集成测试，开发人员则较少这样做。

#### 端到端测试

你可以将*端到端测试*视为功能测试和集成测试的结合。作为另一种黑盒测试，它们检查整个堆栈中的应用程序功能，从前端到后端，在特定环境中运行。这些面向业务的测试应该提供信心，确保整个应用程序仍按预期工作。

端到端测试在特定环境中运行应用程序。通常，许多依赖关系的复杂性增加了不稳定测试的风险，虽然应用程序正常运行，但环境导致测试失败。因此，端到端测试是最耗时的创建和维护测试。由于其复杂性，我们必须谨慎设计它们。在执行过程中，它们通常较慢，容易遇到超时问题，且像几乎所有的黑盒测试一样，无法提供详细的错误报告。因此，它们仅测试最关键的面向业务的场景。通常，质量保证经理编写这些测试。

#### 快照测试

本章前面描述的测试通过一些断言来检查代码。相比之下，*快照测试*则是将应用程序当前的视觉（或用户界面）状态与其之前的版本进行比较。因此，这些测试也称为视觉回归测试。在每个测试中，我们会创建新的快照，然后与之前存储的快照进行比较，这为测试用户界面组件和完整页面提供了一种低成本的方法。我们不再手动创建和维护描述界面每个属性的测试，比如组件的高度、宽度、位置和颜色，而是可以通过快照来包含所有这些属性。

执行这种类型测试的一种方法是创建并比较截图。通常，一个无头浏览器会渲染组件；测试运行器等待页面渲染完成后再捕捉其图像。不幸的是，这个过程相对较慢，并且无头浏览器存在不稳定的情况。Jest 采用了不同的方法进行快照测试。它不依赖无头浏览器和图像文件，而是将 React 用户界面组件渲染到虚拟 DOM 中，进行序列化并将其保存为纯文本的*snap*文件，存储在*__snapshots__*目录下。因此，Jest 的快照测试具有更高的性能，并且更少出错。你将在第二部分构建的 Food Finder 应用中使用快照测试来验证构建的完整性并测试 React 组件。

练习 8：为天气应用添加测试用例

只要你遵循我们讨论过的基本原则，就没有对错之分来测试你的代码。单元测试、快照测试和端到端测试都是你工具包中的不同工具，你必须在编写测试的时间和每种测试的实用性之间找到平衡。关于测试什么内容，也没有共识。虽然你应该努力达到 90%以上的代码覆盖率，但一般的经验法则是，至少覆盖应用程序中最关键的部分进行单元测试，然后编写一些集成测试，以验证你的应用在每次部署时是否能够正常工作。

对于我们的天气应用，我们希望测试用例覆盖四个核心方面。首先，我们将添加单元测试来评估中间件和服务。即使 REST API 端点和 React 用户界面组件可以在浏览器中直接进行测试，我们也会为它们添加测试用例：一个用于用户界面组件的基本快照测试，以及一个针对 REST API 端点*/v1/weather/[zipcode].ts*的端到端测试。

出于简便考虑，我们选择测试 REST 端点而不是 GraphQL API，因为每个 REST 端点都有自己的文件，而所有 GraphQL API 共享一个入口点，这使得测试更为复杂。然而，测试这个 GraphQL API 将是一个很好的练习，帮助你在完成本章后探索端到端测试。

#### 使用间谍测试中间件

连接数据库的中间件是应用程序的核心部分，但我们无法直接访问它，因为它没有暴露任何 API。我们只能通过检查数据库或通过 Mongoose、某个服务或 API 端点运行查询来间接测试它。这些方法都能奏效，但如果我们想将数据库连接作为单元测试进行测试，我们需要尽可能地将该组件隔离开来。

为此，我们将使用 Jest 内置的间谍来验证我们的中间件是否成功调用了建立与 MongoDB 内存服务器连接所需的所有函数。导航到你的 *__tests__* 文件夹，并在其中创建一个新文件夹 *middleware*，然后在其中创建一个文件 *db-connect.test.ts*。然后，将列表 8-19 中的代码复制到该文件中。

```
/**
 * @jest-environment node
 */

import dbConnect from "../../middleware/db-connect";
import mongoose from "mongoose";
import {MongoMemoryServer} from "mongodb-memory-server";

describe("dbConnect ", () => {

    let connection: any;

    afterEach(async () => {
        jest.clearAllMocks();
        await connection.stop();
        await mongoose.disconnect();
    });

    afterAll(async () => {
        jest.restoreAllMocks();
    });

    test("calls MongoMemoryServer.create()", async () => {
        const spy = jest.spyOn(MongoMemoryServer, "create");
        connection = await dbConnect();
        expect(spy).toHaveBeenCalled();
    });

    test("calls mongoose.disconnect()", async () => {
        const spy = jest.spyOn(mongoose, "disconnect");
        connection = await dbConnect();
        expect(spy).toHaveBeenCalled();
    });

    test("calls mongoose.connect()", async () => {
        const spy = jest.spyOn(mongoose, "connect");
        connection = await dbConnect();
        const MONGO_URI = connection.getUri();
        expect(spy).toHaveBeenCalledWith(MONGO_URI, {dbName: "Weather"});
    });

}); 
```

列表 8-19：数据库连接的 __tests__/middleware/db-connect.test.ts 套件

这段代码大部分与本章之前你编写的测试套件相似。但我们现在不是在测试简化的示例代码，而是在测试真实的代码，这要求我们做出一些调整。

首先，我们将 Jest 的测试环境设置为node，该环境模拟了 Node.js 运行时。之后，在编写快照测试时，我们将使用 Jest 的默认环境，称为jsdom，它通过提供一个window对象以及所有常见的 DOM 属性和函数来模拟浏览器。通过始终在文件中设置这些环境，我们避免了因使用错误环境而引发的问题。然后，像往常一样，我们导入所需的包。

现在我们可以开始为dbConnect函数编写测试套件。我们在测试套件的作用域中定义一个connection变量来存储数据库连接，然后我们可以访问 MongoDB 的服务器实例，包括它的方法和属性。例如，我们将使用这些来停止连接并在每次测试后断开与服务器的连接，以确保每个测试用例是独立的。

为了能够存储连接，我们首先需要从文件*db-connect.ts*中的<sup class="SANS_TheSansMonoCd_W5Regular_11">dbConnect</sup>函数返回mongoServer常量。打开文件并在<sup class="SANS_TheSansMonoCd_W5Regular_11">dbConnect</sup>函数的闭合大括号（}）之前添加一行代码return mongoServer。时不时地，你需要修改你之前写的代码，以适应测试的要求。换句话说，你需要调整代码，使其可以进行测试。

现在我们使用刚刚暴露的连接，并设置<sup class="SANS_TheSansMonoCd_W5Regular_11">afterEach</sup>钩子，它在每个测试用例后运行，用于将模拟函数重置为初始模拟状态，从而清除之前收集的所有数据。这是必要的，因为否则间谍会报告在前一次调用中获取的信息，因为它们会在所有测试套件中保留其状态。此外，我们为每个测试用例重新创建数据库连接。因此，在每个测试之后，我们需要停止当前连接并显式断开与数据库的连接。然后，我们设置<sup class="SANS_TheSansMonoCd_W5Regular_11">afterAll</sup>钩子，通过restoreAllMocks函数删除所有模拟并恢复原始函数。

我们的测试用例应该都遵循*arrange, act, assert*模式。在回顾这些用例时，你可能会发现打开*middleware*文件夹中的*db-connect.ts*文件并跟着一起操作会很有帮助。第一个测试用例验证了调用MongoMemoryServer上的<sup class="SANS_TheSansMonoCd_W5Regular_11">create</sup>函数，因为这是我们在*db-connect.ts*文件中调用的第一个函数。为了做到这一点，我们使用jest.spyOn方法创建一个间谍。该方法的参数是一个对象的名称以及要监视的对象方法。然后我们对待测试的代码进行操作，并调用<sup class="SANS_TheSansMonoCd_W5Regular_11">dbConnect</sup>函数。最后，我们断言该间谍已被调用。

第二个测试用例的工作方式类似，不同的是它监听了另一个方法。我们使用它来检查 mongoose.disconnect 是否在执行 dbConnect 时成功调用。第三个测试用例引入了一个新的匹配器。我们不再仅使用 toHaveBeenCalled 来验证调用本身，而是使用 toHaveBeenCalledWith 来验证调用的参数。在这里，我们直接从连接中获取连接字符串并将其存储在变量 MONGO_URI 中。我们还硬编码了要连接的数据库。然后我们调用匹配器，传递预期的参数并验证它们是否符合我们的预期。

现在运行测试套件，使用 npm test。所有测试应当通过，并且达到 100% 的测试覆盖率。

#### 创建用于测试服务的模拟

虽然我们为中间件编写的测试非常简单，但服务测试稍微复杂一些。如果你打开 *mongoose/weather/services.ts* 文件，你会发现这些服务依赖于 WeatherModel，它是 Mongoose 访问 MongoDB 集合的网关。每个服务调用模型上的一个方法，而该方法又需要一个数据库连接。我们在这里不重新评估这些数据库连接；相反，这个测试套件的目标是验证服务函数是否调用了正确的 WeatherModel 函数。为此，我们将创建一个模拟的 WeatherModel，它暴露与模拟函数相同的一组 API。

我们首先编写模拟的模型。按照惯例，我们创建了文件 *mongoose/weather/__mocks__/model.ts* 并添加了 Listing 8-20 中的代码。

```
import {WeatherInterface} from "../interface";

type param = {
    [key: string]: string;
};

const WeatherModel = {
    create: jest.fn((newData: WeatherInterface) => Promise.resolve(true)),
    findOne: jest.fn(({zip: paramZip}: param) => Promise.resolve(true)),
    updateOne: jest.fn(({zip: paramZip}: param, newData: WeatherInterface) =>
        Promise.resolve(true)
    ),
    deleteOne: jest.fn(({zip: paramZip}: param) => Promise.resolve(true))
};
export default WeatherModel; 
```

Listing 8-20: WeatherModel 的模拟

我们实现了 WeatherInterface 并定义了新的 param 类型，这是一个包含键值对的对象，用于给第一个参数指定类型。我们将模拟的 WeatherModel 设置为默认导出，并使用一个实现了实际 WeatherModel 四个方法的对象，每个方法的参数与原始方法相同。它们还采用了原始 Mongoose 模型的方法。因为它们是异步函数，我们返回一个解析为 true 的 Promise。

现在我们可以为服务编写测试套件。它们检查每个服务在成功时返回 true，并调用模拟的 WeatherModel 的正确方法。创建文件 */__tests__/mongoose/weather/services.test.ts*，并将 列表 8-21 中的代码添加到该文件中。

```
/**
 * @jest-environment node
 */
import {WeatherInterface} from "../../../mongoose/weather/interface";
import {
    findByZip,
    storeDocument,
    updateByZip,
    deleteByZip,
} from "../../../mongoose/weather/services";

import WeatherModel from "../../../mongoose/weather/model";
jest.mock("../../../mongoose/weather/model");

describe("the weather services", () => {

    let doc: WeatherInterface = {
        zip: "test",
        weather: "weather",
        tempC: "00",
        tempF: "01",
        friends: []
    };

 afterEach(async () => {
        jest.clearAllMocks();
    });

    afterAll(async () => {
        jest.restoreAllMocks();
    });

    describe("API storeDocument", () => {
        test("returns true", async () => {
            const result = await storeDocument(doc);
            expect(result).toBeTruthy();
        });
        test("passes the document to Model.create()", async () => {
            const spy = jest.spyOn(WeatherModel, "create");
            await storeDocument(doc);
            expect(spy).toHaveBeenCalledWith(doc);
        });
    });

    describe("API findByZip", () => {
        test("returns true", async () => {
            const result = await findByZip(doc.zip);
            expect(result).toBeTruthy();
        });
        test("passes the zip code to Model.findOne()", async () => {
            const spy = jest.spyOn(WeatherModel, "findOne");
            await findByZip(doc.zip);
            expect(spy).toHaveBeenCalledWith({zip: doc.zip});
        });
    });

    describe("API updateByZip", () => {
        test("returns true", async () => {
            const result = await updateByZip(doc.zip, doc);
            expect(result).toBeTruthy();
        });
        test("passes the zip code and the new data to Model.updateOne()", async () => {
            const spy = jest.spyOn(WeatherModel, "updateOne");
            const result = await updateByZip(doc.zip, doc);
            expect(spy).toHaveBeenCalledWith({zip: doc.zip}, doc);
        });
    });

    describe("API deleteByZip", () => {
        test("returns true", async () => {
            const result = await deleteByZip(doc.zip);
            expect(result).toBeTruthy();
        });
        test("passes the zip code Model.deleteOne()", async () => {
            const spy = jest.spyOn(WeatherModel, "deleteOne");
            const result = await deleteByZip(doc.zip);
            expect(spy).toHaveBeenCalledWith({zip: doc.zip});
        });
    });

}); 
```

列表 8-21：__tests__/mongoose/weather/services.test.ts 中更新的测试套件

与之前的测试套件一样，我们首先设置环境并导入模块。我们还导入了 WeatherModel，并使用 jest.mock 调用我们创建的模拟模型路径，从而有效地替换了测试代码中的原始模型。然后，我们创建一个包含一些测试数据的文档。我们将其存储在常量 doc 中，并将其传递给模拟模型的方法。与之前一样，我们使用 afterEach 钩子在每个测试后重置所有模拟，使用 afterAll 钩子在所有测试用例完成后移除模拟并恢复原始函数。

我们为四个服务创建了一个嵌套的测试套件。每个服务都有相同的两个单元测试：一个是使用 toBeTruthy 匹配器验证成功时的返回值，另一个是间谍（spy）监视一个特定的 WeatherModel 模拟函数。代码遵循与前一个测试套件相同的模式，并使用相同的匹配器。

我们在运行 npm test 后收到的代码覆盖率报告显示，我们测试了大约 70% 的服务代码。如果你查看最后一列中列出的未覆盖行，你会看到它们包含了 console.log(err); 输出。这个输出会在异步调用模型方法失败时使用：

```
PASS  __tests__/mongoose/weather/services.test.ts
PASS  __tests__/middleware/dbconnect.test.ts (7.193 s)

--------------------|---------|----------|---------|---------|-------------------
File                | % Stmts | % Branch | % Funcs | % Lines | Uncovered Lines
--------------------|---------|----------|---------|---------|-------------------
All files           |   83.63 |      100 |   88.23 |   82.35 |
 middleware         |     100 |      100 |     100 |     100 |
  db-connect.test.ts|     100 |      100 |     100 |     100 |
 mongoose/weather.  |   77.41 |      100 |     100 |   75.86 |
  services.test.ts  |   70.83 |      100 |     100 |   70.83 |8,20-22,33-35,43-45
--------------------|---------|----------|---------|---------|------------------- 
```

本章的目的，我们将保留这些未覆盖的行。否则，我们可以修改模拟的模型，使其抛出一个错误——例如，提供一个无效的文档——然后为每个服务添加一个第三个测试案例，验证错误。

#### 执行 REST API 的端到端测试

高级 API 测试可能会使用专门的 API 测试库，如 *SuperTest*，它提供了 HTTP 状态码的匹配器，并简化了请求和响应的处理。或者，它们可能会使用像 Postman 这样的 GUI 工具。在本例中，我们将仅通过使用本地的 fetch 方法测试返回的数据是否符合我们的预期。

与之前的测试不同，这次的测试并没有隔离任何单一组件，因为我们的目标是验证系统的所有组件是否按预期协同工作。为了检查当提供输入时，API 是否从数据库返回正确的响应，我们的端到端测试将做出一些假设：所有层级已经独立测试过，数据库包含初始种子数据，并且我们的应用运行在*http://localhost:3000/*。

为了验证我们的第一个假设，打开 API 端点文件*pages/api/v1/weather/[zipcode].ts*。你会注意到，API 代码导入了两个函数，来自服务模块的findByZip和中间件的dbConnect，这两个我们已经测试过的函数。第二个假设也得到了验证；数据库在每次启动时都会加载初始种子数据。创建文件*zipcode.e2e .test.ts*，路径为*__tests__/pages/api/v1/weather/*，并添加清单 8-22 中的代码。

```
/**
 * @jest-environment node
 */

describe("The API /v1/weather/[zipcode]", () => {
    test("returns the correct data for the zipcode 96815", async () => {
        const zip = "96815";
        let response = await fetch(`http://localhost:3000/api/v1/weather/${zip}`);
        let body = await response.json();
        expect(body.zip).toEqual(zip);
    });
});

export {}; 
```

清单 8-22：REST API 的测试套件

我们将环境设置为node，然后定义一个包含一个测试用例的测试套件。在该测试用例中，我们提供一个与初始种子数据集匹配的邮政编码。然后，我们使用自 Node.js 版本 17.5 起可用的原生fetch方法，调用本地的天气 API，并检查返回的邮政编码是否与作为参数传入的邮政编码相同。我们添加一个空的 export 语句，定义此文件为一个 ES6 模块。

测试应该通过，并且代码覆盖率为 100%。现在我们确信应用的核心功能按预期工作，可以开始测试用户界面组件。

使用 fetch 时，你可能会遇到两个常见的错误信息。第一个，ECONNREFUSED，表示 fetch 无法连接到你的应用程序，因为它没有运行。使用 npm run dev 启动应用程序，或者如果你不使用端口 3000，可以调整 fetch 调用中的端口。第二个错误提示测试超出了 5,000 毫秒的超时时间。如果你为了测试启动了应用程序，并且没有使用先前运行的应用程序，Next.js 会在测试消耗 API 路由时立即编译它。根据你的环境，这可能需要比默认超时时间更长。将 jest.setTimeout(20000); 这一行添加到文件顶部的 describe 方法之前，以增加超时时间，将测试等待时间从 5,000 毫秒增加到 20,000 毫秒。

#### 通过快照测试评估用户界面

快照测试验证页面渲染的 HTML 在两次测试运行之间没有变化。为了使用 Jest 实现这一点，我们必须首先准备好环境。将 jsdom 环境、react-testing-library 和 react-test-renderer 添加到项目中：

```
$ **npm install --save-dev jest-environment-jsdom**
$ **npm install --save-dev @testing-library/react @testing-library/jest-dom**
$ **npm install --save-dev @types/react-test-renderer react-test-renderer** 
```

我们需要这些工具来模拟浏览器环境，并在测试用例中渲染 React 组件。现在我们将相应地修改根目录下的 *jest.config.js* 文件。用 Listing 8-23 中的代码替换它的内容。

```
const nextJest = require("next/jest");
const createJestConfig = nextJest({});

module.exports = createJestConfig(nextJest({})); 
```

Listing 8-23: 更新后的 jest.config.js 文件

这段代码导入了 *next/jest* 包，并导出了一个具有 Next.js 项目默认属性的 Jest 配置。这是最简单的 Next.js 兼容 Jest 配置形式。如果你查看官方的 Next.js 设置指南 [*https://<wbr>nextjs<wbr>.org<wbr>/docs<wbr>/testing*](https://nextjs.org/docs/testing)，你会看到它概述了一些基本的配置选项，但我们不需要这些选项。

##### 第一个版本

快照测试渲染一个组件或页面，拍摄其快照并将其作为序列化的 JSON 存储在与测试套件并列的*__snapshots__*文件夹中。在每次连续运行时，Jest 会将当前的快照与存储的参考快照进行对比。只要它们相同，快照测试就通过。要生成初始快照，创建一个新的文件夹，*__tests__/pages/components*，以及文件*weather.snapshot.test.tsx*，然后将 Listing 8-24 中的代码添加进去。

```
/**
 * @jest-environment node
 */

import {act, create} from "react-test-renderer";
import PageComponentWeather from "../../../pages/components/weather";

describe("PageComponentWeather", () => {
    test("renders correctly", async () => {
        let component: any;
        await act(async () => {
            component =
                await create(<PageComponentWeather></PageComponentWeather>);
        });
        expect(component.toJSON()).toMatchSnapshot();
    });
}); 
```

列表 8-24：PageComponentWeather 的快照测试

我们的快照测试的前几行设置了环境为 jsdom，并导入了测试渲染器的 act 和 create 方法，用于测试 React 组件，我们将在下一行导入它们。

接下来，我们编写模拟的用户行为，并将组件的创建包裹在异步的 act 函数中。正如你可能猜到的，这个函数的名字来源于 *arrange, act, assert* 模式，确保在继续进行测试用例之前，所有相关的 DOM 更新已经应用。对于所有导致 React 状态更新的语句，它都是必须的，在这里，它延迟了测试执行，直到 useEffect 钩子执行完毕。

然后我们编写一个测试用例，等待 create 函数，该函数渲染 JSX 组件。这让我们可以在模拟的浏览器环境中生成 HTML，并将结果存储在变量中。我们等待组件的渲染，以便在继续测试用例之前，HTML 可以用于我们的后续交互。接着我们将渲染的组件序列化为 JSON 字符串，并使用一个新的匹配器 toMatchSnapshot，它将当前的 JSON 字符串与存储的参考值进行比较。

一次试运行显示所有测试都成功。我们看到两件有趣的事情——测试创建了一个快照，并且我们达到了 81% 的测试覆盖率：

```
PASS  __tests__/mongoose/weather/services.test.ts
PASS  __tests__/pages/api/v1/weather/zipcode.e2e.test.ts
PASS  __tests__/middleware/dbconnect.test.ts (7.193 s)
PASS  __tests__/pages/components/weather.snapshot.test.tsx

---------------------|---------|----------|---------|---------|-------------------
File                 | % Stmts | % Branch | % Funcs | % Lines | Uncovered Lines
---------------------|---------|----------|---------|---------|-------------------
All files            |   83.63 |      100 |   88.23 |   82.35 |
 middleware          |     100 |      100 |     100 |     100 |
  db-connect.test.ts |     100 |      100 |     100 |     100 |
 mongoose/weather    |   77.41 |      100 |     100 |   75.86 |
  services.test.ts   |   70.83 |      100 |     100 |   70.83 |8,20-22,33-35,43-45
 pages/api/v1/       |         |          |         |         |
  weather            |         |          |         |         |
    [zipcode].ts     |     100 |      100 |     100 |     100 |
 pages/components    |   81.81 |      100 |      60 |      80 |
  weather.tsx        |   81.81 |      100 |      60 |      80 |8,12
---------------------|---------|----------|---------|---------|-------------------
Snapshot Summary
 › 1 snapshot written from 1 test suite. 
```

你可以通过打开 *weather.snapshot.test.tsx.snap* 文件，在 *__snapshots__* 文件夹中查看创建的快照。它应该与 列表 8-25 中的代码非常相似，你会发现它不过是将渲染的 HTML 保存为多行模板字面量。你的 HTML 可能与这里展示的内容不完全相同；重要的是，在每次测试运行后，当 react-test-renderer 渲染组件时，它应该看起来相同。

```
// Jest Snapshot v1, https://goo.gl/fbAQLP

exports[`PageComponentWeather renders correctly 1`] = `
<h1
    data-testid="h1"
    onClick={[Function]}
>
    The weather is
    sunny
    , and the counter shows
    0
</h1>
`; 
```

列表 8-25：包含序列化 HTML 的 weather.snapshot.test.tsx.snap 文件

我们还看到计数器被设置为 0，这表明 useEffect 钩子在我们创建快照之前并没有运行。如果你打开组件的文件并检查未覆盖的行，你会发现这些行与增加 state 变量的点击处理器相关，正如我们预期的那样，还有 useEffect 钩子。我们也想测试这些核心功能。

##### 第二个版本

我们将修改测试代码，以覆盖之前未测试的功能。将 清单 8-26 中的代码粘贴到快照测试文件中。

```
/**
 * @jest-environment node
 */

import {act, create} from "react-test-renderer";
import PageComponentWeather from "../../../pages/components/weather";

describe("PageComponentWeather", () => {
    test("renders correctly", async () => {
        let component: any;
        await act(async () => {
            component = await create(<PageComponentWeather></PageComponentWeather>);
        });
        expect(component.toJSON()).toMatchSnapshot();
    });

    test("clicks the h1 element and updates the state", async () => {
        let component: any;
        await act(async () => {
            component = await create(<PageComponentWeather></PageComponentWeather>);
            component.root.findByType("h1").props.onClick();
        });
        expect(component.toJSON()).toMatchSnapshot();
    });

}); 
```

清单 8-26：更新后的快照测试

在更新后的代码中，我们添加了另一个测试用例，它找到页面上的标题并模拟用户点击它。记住在前面的章节中，这会增加状态变量 counter。再次提醒，我们等待组件的创建，并使用 act 函数。

如果你重新运行测试，应该会看到失败。测试运行器告诉我们快照不匹配：

```
FAIL  __tests__/pages/components/weather.snapshot.test.tsx
  • PageComponentWeather › renders correctly
`--snip--`
 › 1 snapshot failed.
`--snip--`
Snapshot Summary
 › 1 snapshot failed from 1 test suite.
› Inspect your code changes or run `npm test -- -u` to update them. 
```

因为我们修改了测试用例，等待 useEffect 钩子，并将状态变量 counter 设置为 1 而不是 0，所以 DOM 也发生了变化。按照测试运行器的建议，使用 npm test -- -u 重新运行测试，创建一个新的更新后的快照。现在测试应该会成功，报告我们组件的测试覆盖率为 100%。

尝试运用你新学到的知识。例如，你能为 *pages* 目录中的页面路由编写快照测试，或者为 GraphQL API 编写一组端到端测试吗？

### 总结

现在你应该能够使用 Jest 创建自动化测试，并且更广泛地说，自己设计一个测试计划，以在努力和回报之间找到平衡。我们讨论了 TDD 和单元测试的好处，然后使用 *arrange, act, assert* 模式，按照测试驱动原则开发了一个简单的 sum 函数。接着，我们使用三种类型的测试替代品，在计算 Fibonacci 数列时替换了 sum 函数。最后，我们向现有的 Next.js 应用程序添加了单元和快照测试，创建了一个 Mongoose 模型的模拟，并使用间谍验证了我们的假设。

要了解更多关于 Jest 和自动化测试的信息，请查阅官方 Jest 文档：[*https://<wbr>jestjs<wbr>.io<wbr>/docs<wbr>/getting<wbr>-started*](https://jestjs.io/docs/getting-started)。在下一章中，你将探索授权和认证之间的区别，以及如何在应用程序中利用 OAuth。
