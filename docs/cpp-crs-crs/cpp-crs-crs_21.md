## 第二十一章：**算法**

*这才是编程的精髓。通过将一个复杂的想法拆解成小步骤，甚至是一个愚蠢的机器也能处理时，你自己已经学到了关于它的某些东西。*

—**道格拉斯·亚当斯**，《Dirk Gently 的全息侦探事务所》

![图片](img/common.jpg)

*算法*是一种解决一类问题的过程。std 库和 Boost 库包含了大量你可以在程序中使用的算法。因为许多聪明的人花了大量时间来确保这些算法的正确性和效率，所以你通常不需要尝试自己编写排序算法等。

由于本章涵盖了几乎整个 std 库算法集，因此篇幅较长；然而，每个算法的介绍都很简洁。首次阅读时，你应该浏览每一节，了解可以使用的各种算法。不要试图记住它们。相反，应该专注于获得对未来编写代码时，能通过它们解决哪些问题的洞察。这样，当你需要使用某个算法时，你可以说：“等等，难道不是有人已经发明过这个轮子了吗？”

在开始使用算法之前，你需要对复杂度和并行性有所了解。这两种算法特性是决定你的代码性能的主要因素。

### 算法复杂度

*算法复杂度*描述了计算任务的难度。一种量化这种复杂度的方法是使用*巴赫曼-兰道*或*“大 O”表示法*。大 O 表示法根据计算随着输入大小的变化情况来描述函数。此表示法仅包括复杂度函数的主项。*主项*是输入大小增加时增长最快的项。

例如，复杂度大约随着每个额外输入元素增加一个固定值的算法，其 Big O 表示法为**O(N)**，而复杂度在额外输入下不会变化的算法，其 Big O 表示法为**O(1)**。

本章描述了 std 库算法，它们属于以下五个复杂度类别。为了让你了解这些算法如何扩展，每个类别都列出了其 Big O 符号以及输入从 1,000 个元素增加到 10,000 个元素时，因主项而需要的大致额外操作次数。每个示例提供了一个具有给定复杂度类别的操作，其中*N*是涉及该操作的元素数量：

**常数时间 O(1)** 无需额外计算。一个例子是确定`std::vector`的大小。

**对数时间 O(log *N*)** 大约需要进行一次额外的计算。一个例子是查找`std::set`中的元素。

**线性时间 O(*N*)** 大约需要 9,000 次额外计算。一个例子是对集合中的所有元素求和。

**准线性时间 O(*N* log *N*)** 大约增加 37,000 次计算。一个例子是快速排序，常用的排序算法。

**多项式时间（或二次时间）O(*N*²)** 大约增加 99,000,000 次计算。一个例子是将一个集合中的所有元素与另一个集合中的所有元素进行比较。

计算机科学的一个完整领域致力于根据计算问题的难度来对其进行分类，因此这是一个复杂的话题。本章提到的每个算法的复杂度取决于目标序列的大小如何影响所需工作量。实际上，你应该对性能进行分析，以确定某个算法是否具备合适的扩展性。但这些复杂度类别可以让你大致了解某个算法的开销。

### 执行策略

一些算法，通常被称为*并行算法*，可以将一个算法分解，使得独立的实体可以同时在不同部分解决问题。许多标准库算法允许你通过*执行策略*来指定并行性。执行策略表示算法允许的并行度。从标准库的角度看，算法可以按*顺序*执行或*并行*执行。顺序算法一次只能由单个实体处理问题；并行算法可以有多个实体共同协作解决问题。

此外，并行算法可以是*向量化*的或*非向量化*的。向量化算法允许实体以未指定的顺序执行工作，甚至允许单个实体同时处理问题的多个部分。例如，需要在实体之间进行同步的算法通常是不可向量化的，因为同一实体可能会多次尝试获取锁，导致死锁。

`<execution>`头文件中存在三种执行策略：

+   `std::execution::seq`指定顺序执行（非并行执行）。

+   `std::execution::par`指定并行执行。

+   `std::execution::par_unseq`指定并行*且*向量化的执行。

对于那些支持执行策略的算法，默认策略是`seq`，这意味着你必须显式选择并行执行及其相关的性能优势。请注意，C++标准没有明确指定这些执行策略的具体含义，因为不同平台处理并行性的方式不同。当你提供非顺序执行策略时，你仅仅是在声明“这个算法是安全的，可以并行化”。

在第一章中，你将更详细地探讨执行策略。目前，只需注意一些算法允许并行性。

**警告**

*本章中的算法描述并不完整。它们包含足够的信息，能够为您提供有关标准库中许多可用算法的良好背景。我建议您在确定适合自己需求的算法后，查阅本章末尾的“进一步阅读”部分中的资源。接受可选执行策略的算法，在提供非默认策略时，通常会有不同的要求，特别是在涉及迭代器时。例如，如果一个算法通常接受输入迭代器，使用执行策略通常会导致该算法要求使用前向迭代器。列出这些差异会使已经相当庞大的章节更长，因此描述中省略了这些差异。*

**如何使用本章**

本章是一本快速参考，包含 50 多个算法。每个算法的覆盖面简洁明了。每个算法以简短的描述开始，紧接着是该算法的函数声明的简写表示，并附有每个参数的解释。声明中用括号表示可选参数。接下来，列出了算法的复杂度。最后是一个非详尽但具有说明性的示例，展示了该算法的应用。本章中的几乎所有示例都是单元测试，隐含地包括以下前言：

```
#include "catch.hpp"
#include <vector>
#include <string>

using namespace std;
```

如有需要，参阅相关小节[算法]获取详细信息。

### 非修改序列操作

*非修改序列操作*是一个在序列上执行计算但不以任何方式修改序列的算法。您可以将这些算法视为`const`算法。本节中解释的每个算法都在`<algorithm>`头文件中。

#### *all_of*

`all_of`算法用于判断序列中的每个元素是否符合用户指定的某些标准。

如果目标序列为空，或者`pred`对序列中的*所有*元素返回`true`，则算法返回`true`；否则，返回`false`。

```
bool all_of([ep], ipt_begin, ipt_end, pred);
```

##### 参数

+   一个可选的`std::execution`执行策略`ep`（默认：`std::execution::seq`）

+   一对`InputIterator`对象，`ipt_begin`和`ipt_end`，表示目标序列

+   一个一元谓词`pred`，接受目标序列中的一个元素

##### 复杂度

**线性** 该算法最多调用`pred``distance(ipt_begin, ipt_end)`次。

##### 示例

```
#include <algorithm>

TEST_CASE("all_of") {
  vector<string> words{ "Auntie", "Anne's", "alligator" }; ➊
  const auto starts_with_a =
    [](const auto& word➋) {
      if (word.empty()) return false; ➌
      return word[0] == 'A' || word[0] == 'a'; ➍
    };
  REQUIRE(all_of(words.cbegin(), words.cend(), starts_with_a)); ➎
  const auto has_length_six = [](const auto& word) {
    return word.length() == 6; ➏
  };
  REQUIRE_FALSE(all_of(words.cbegin(), words.cend(), has_length_six)); ➐
}
```

在构造一个包含`string`对象的`vector`，名为`words` ➊之后，您构造了一个名为`starts_with_a`的 lambda 谓词，它接受一个名为`word`的单一对象 ➋。如果`word`为空，`starts_with_a`返回`false` ➌；否则，如果`word`以`a`或`A`开头，返回`true` ➍。由于所有的`word`元素都以`a`或`A`开头，当应用`starts_with_a`时，`all_of`返回`true` ➎。

在第二个例子中，你构造了谓词`has_length_six`，只有当`word`的长度为六时，它才返回`true` ➏。因为`alligator`的长度不是六，`all_of`在应用`has_length_six`到`words`时返回`false` ➐。

#### *any_of*

`any_of`算法判断序列中是否有任何元素满足用户指定的标准。

如果目标序列为空，或`pred`对于序列中的*任何*元素为`true`，则算法返回`false`；否则返回`false`。

```
bool any_of([ep], ipt_begin, ipt_end, pred);
```

##### 参数

+   一个可选的`std::execution`执行策略，`ep`（默认值：`std::execution::seq`）

+   一对`InputIterator`对象，`ipt_begin`和`ipt_end`，表示目标序列

+   一个一元谓词`pred`，接受来自目标序列的一个元素

##### 复杂度

**线性** 算法最多调用`pred` `distance(ipt_begin, ipt_end)`次。

##### 示例

```
#include <algorithm>

TEST_CASE("any_of") {
  vector<string> words{ "Barber", "baby", "bubbles" }; ➊
  const auto contains_bar = [](const auto& word) {
    return word.find("Bar") != string::npos;
  }; ➋
  REQUIRE(any_of(words.cbegin(), words.cend(), contains_bar)); ➌

  const auto is_empty = [](const auto& word) { return word.empty(); }; ➍
  REQUIRE_FALSE(any_of(words.cbegin(), words.cend(), is_empty)); ➎
}
```

在构造了一个包含`string`对象的`vector`，命名为`words` ➊之后，你构造了一个名为`contains_bar`的 lambda 谓词，它接受一个名为`word`的单一对象 ➋。如果`word`包含子串`Bar`，它返回`true`；否则返回`false`。因为`Barber`包含`Bar`，`any_of`在应用`contains_bar`时返回`true` ➌。

在第二个例子中，你构造了谓词`is_empty`，只有当`word`为空时，它才返回`true` ➍。因为没有任何单词为空，`any_of`在应用`is_empty`到`words`时返回`false` ➎。

#### *none_of*

`none_of`算法判断序列中是否没有任何元素满足用户指定的标准。

如果目标序列为空，或`pred`对于序列中的*任何*元素为`true`，则算法返回`true`；否则返回`false`。

```
bool none_of([ep], ipt_begin, ipt_end, pred);
```

##### 参数

+   一个可选的`std::execution`执行策略，`ep`（默认值：`std::execution::seq`）

+   一对`InputIterator`对象，`ipt_begin`和`ipt_end`，表示目标序列

+   一个一元谓词`pred`，接受来自目标序列的一个元素

##### 复杂度

**线性** 算法最多调用`pred` `distance(ipt_begin, ipt_end)`次。

##### 示例

```
#include <algorithm>

TEST_CASE("none_of") {
  vector<string> words{ "Camel", "on", "the", "ceiling" }; ➊
  const auto is_hump_day = [](const auto& word) {
    return word == "hump day";
  }; ➋
  REQUIRE(none_of(words.cbegin(), words.cend(), is_hump_day)); ➌

  const auto is_definite_article = [](const auto& word) {
    return word == "the" || word == "ye";
  }; ➍
  REQUIRE_FALSE(none_of(words.cbegin(), words.cend(), is_definite_article)); ➎
}
```

在构造了一个包含`string`对象的`vector`，命名为`words` ➊之后，你构造了一个名为`is_hump_day`的 lambda 谓词，它接受一个名为`word`的单一对象 ➋。如果`word`等于`hump day`，它返回`true`；否则返回`false`。因为`words`中不包含`hump day`，所以`none_of`在应用`is_hump_day`时返回`true` ➌。

在第二个例子中，你构造了谓词`is_definite_article`，只有当`word`是定冠词时，它才返回`true` ➍。因为`the`是定冠词，`none_of`在应用`is_definite_article`到`words`时返回`false` ➎。

#### *for_each*

`for_each`算法对序列中的每个元素应用某个用户定义的函数。

该算法对目标序列的每个元素应用 `fn`。虽然 `for_each` 被认为是一个不修改序列的操作，如果 `ipt_begin` 是一个可变迭代器，`fn` 可以接受一个非 `const` 参数。`fn` 返回的任何值都会被忽略。

如果省略了 `ep`，`for_each` 将返回 `fn`。否则，`for_each` 返回 `void`。

```
for_each([ep], ipt_begin, ipt_end, fn);
```

##### 参数

+   一个可选的 `std::execution` 执行策略，`ep`（默认值：`std::execution::seq`）

+   一对 `InputIterator` 对象，`ipt_begin` 和 `ipt_end`，表示目标序列

+   一个一元函数，`fn`，接受目标序列中的一个元素

##### 复杂度

**线性** 该算法恰好调用 `fn` `distance(ipt_begin, ipt_end)` 次。

##### 附加要求

+   如果省略了 `ep`，`fn` 必须是可移动的。

+   如果提供了 `ep`，`fn` 必须是可复制的。

##### 示例

```
#include <algorithm>

TEST_CASE("for_each") {
  vector<string> words{ "David", "Donald", "Doo" }; ➊
  size_t number_of_Ds{}; ➋
  const auto count_Ds = &number_of_Ds➌ {
    if (word.empty()) return; ➎
    if (word[0] == 'D') ++number_of_Ds; ➏
  };
  for_each(words.cbegin(), words.cend(), count_Ds); ➐
  REQUIRE(3 == number_of_Ds); ➑
}
```

在构建一个包含 `string` 对象的 `vector`，名为 `words` ➊ 和一个计数器变量 `number_of_Ds` ➋ 后，构建捕获 `number_of_Ds` 引用的 lambda 谓词 `count_Ds` ➌，并接收一个名为 `word` ➍ 的单一对象。如果 `word` 为空，则返回 ➎；否则，如果 `word` 的第一个字母是 `D`，则递增 `number_of_Ds` ➏。

接下来，使用 `for_each` 遍历每个单词，将每个单词传递给 `count_Ds` ➐。结果是 `number_of_Ds` 为三 ➑。

#### *for_each_n*

`for_each_n` 算法对序列中的每个元素应用某个用户定义的函数。

该算法对目标序列的每个元素应用 `fn`。虽然 `for_each_n` 被认为是一个不修改序列的操作，如果 `ipt_begin` 是一个可变迭代器，`fn` 可以接受一个非 `const` 参数。`fn` 返回的任何值都会被忽略。它返回 `ipt_begin+n`。

```
InputIterator for_each_n([ep], ipt_begin, n, fn);
```

##### 参数

+   一个可选的 `std::execution` 执行策略，`ep`（默认值：`std::execution::seq`）

+   一个 `InputIterator` `ipt_begin`，表示目标序列的第一个元素

+   一个整数 `n`，表示期望的迭代次数，以便表示目标序列的半开区间为 `ipt_begin` 到 `ipt_begin+n`（`Size` 是 `n` 的模板类型）。

+   一个一元函数 `fn`，接受目标序列中的一个元素

##### 复杂度

**线性** 该算法恰好调用 `fn` `n` 次。

##### 附加要求

+   如果省略了 `ep`，`fn` 必须是可移动的。

+   如果提供了 `ep`，`fn` 必须是可复制的。

+   `n` 必须是非负数。

##### 示例

```
#include <algorithm>

TEST_CASE("for_each_n") {
  vector<string> words{ "ear", "egg", "elephant" }; ➊
  size_t characters{}; ➋
  const auto count_characters = &characters➌ {
    characters += word.size(); ➎
  };
  for_each_n(words.cbegin(), words.size(), count_characters); ➏
  REQUIRE(14 == characters); ➐
}}
```

在构建一个包含 `string` 对象的 `vector`，名为 `words` ➊ 和一个计数器变量 `characters` ➋ 后，构建捕获 `characters` 引用的 lambda 谓词 `count_characters` ➌，并接收一个名为 `word` ➍ 的单一对象。lambda 将 `word` 的长度加到 `characters` 上 ➎。

接下来，使用 `for_each_n` 遍历每个单词，将每个单词传递给 `count_characters` ➏。结果是 `characters` 为 `14` ➐。

#### *find, find_if, 和 find_if_not*

`find`、`find_if` 和 `find_if_not` 算法查找序列中第一个匹配某些用户定义标准的元素。

这些算法返回指向目标序列中第一个匹配`value`元素的`InputIterator`（`find`），在与`pred`一起调用时返回`true`（`find_if`），或者在与`pred`一起调用时返回`false`（`find_if_not`）。

如果算法未找到匹配项，则返回`ipt_end`。

```
InputIterator find([ep], ipt_begin, ipt_end, value);
InputIterator find_if([ep], ipt_begin, ipt_end, pred);
InputIterator find_if_not([ep], ipt_begin, ipt_end, pred);
```

##### 参数

+   一个可选的`std::execution`执行策略，`ep`（默认值：`std::execution::seq`）

+   一对`InputIterator`对象，`ipt_begin`和`ipt_end`，表示目标序列

+   一个与目标序列的底层类型（`find`）相等可比较的`const`引用`value`，或者一个接受目标序列底层类型作为单一参数的谓词（`find_if` 和 `find_if_not`）

##### 复杂度

**线性** 该算法最多进行`distance(ipt_begin, ipt_end)`次比较（`find`）或调用`pred`（`find_if` 和 `find_if_not`）。

##### 示例

```
#include <algorithm>

TEST_CASE("find find_if find_if_not") {
  vector<string> words{ "fiffer", "feffer", "feff" }; ➊
  const auto find_result = find(words.cbegin(), words.cend(), "feff"); ➋
  REQUIRE(*find_result == words.back()); ➌

  const auto defends_digital_privacy = [](const auto& word) {
    return string::npos != word.find("eff"); ➍
  };

  const auto find_if_result = find_if(words.cbegin(), words.cend(),
                                      defends_digital_privacy); ➎
  REQUIRE(*find_if_result == "feffer"); ➏

  const auto find_if_not_result = find_if_not(words.cbegin(), words.cend(),
                                              defends_digital_privacy); ➐
  REQUIRE(*find_if_not_result == words.front()); ➑
}
```

在构造一个包含`string`对象的`vector`，命名为`words` ➊之后，你使用`find`来定位`feff` ➋，它位于`words`的末尾 ➌。接下来，你构造了谓词`defends_digital_privacy`，如果`word`包含字母`eff` ➍，则返回`true`。然后你使用`find_if`来定位`words`中第一个包含`eff`的字符串 ➎，即`feffer` ➏。最后，你使用`find_if_not`将`defends_digital_privacy`应用于`words` ➐，它返回第一个元素`fiffer`（因为它不包含`eff`） ➑。

#### *find_end*

`find_end`算法查找子序列的最后一次出现。

如果算法未找到符合条件的序列，则返回`fwd_end1`。如果`find_end`确实找到了一个子序列，则返回一个`ForwardIterator`，指向最后一个匹配子序列的第一个元素。

```
InputIterator find_end([ep], fwd_begin1, fwd_end1,
                       fwd_begin2, fwd_end2, [pred]);
```

##### 参数

+   一个可选的`std::execution`执行策略，`ep`（默认值：`std::execution::seq`）

+   两对`ForwardIterator`，`fwd_begin1` / `fwd_end1`和`fwd_begin2` / `fwd_end2`，表示目标序列 1 和 2

+   一个可选的二元谓词`pred`，用于比较两个元素是否相等

##### 复杂度

**二次** 该算法最多进行以下次数的比较或调用`pred`：

```
distance(fwd_begin2, fwd_end2) * (distance(fwd_begin1, fwd_end1) -
                                  distance(fwd_begin2, fwd_end2) + 1)
```

##### 示例

```
#include <algorithm>

TEST_CASE("find_end") {
  vector<string> words1{ "Goat", "girl", "googoo", "goggles" }; ➊
  vector<string> words2{ "girl", "googoo" }; ➋
  const auto find_end_result1 = find_end(words1.cbegin(), words1.cend(),
                                         words2.cbegin(), words2.cend()); ➌
  REQUIRE(*find_end_result1 == words1[1]); ➍

  const auto has_length = [](const auto& word, const auto& len) {
    return word.length() == len; ➎
  };
  vector<size_t> sizes{ 4, 6 }; ➏
  const auto find_end_result2 = find_end(words1.cbegin(), words1.cend(),
                                         sizes.cbegin(), sizes.cend(),
                                         has_length); ➐
  REQUIRE(*find_end_result2 == words1[1]); ➑
}
```

在构造一个包含`string`对象的`vector`，命名为`words1` ➊，另一个名为`words2` ➋之后，你调用`find_end`来确定`words1`中哪个元素开始匹配`words2`的子序列 ➌。结果是`find_end_result1`，其值为`girl` ➍。

接下来，你构造了一个 lambda 表达式`has_length`，它接受两个参数`word`和`len`，如果`word.length()`等于`len` ➎，则返回`true`。你构造了一个名为`sizes`的`size_t`类型的`vector` ➏，并用`words1`、`sizes`和`has_length`调用`find_end` ➐。结果`find_end_result2`指向`words1`中第一个长度为`4`的元素，后面的单词长度为`6`。由于`girl`的长度为`4`，`googoo`的长度为`6`，所以`find_end_result2`指向`girl` ➑。

#### *find_first*

`find_first_of`算法查找序列 1 中第一个等于序列 2 中某个元素的位置。

如果提供了`pred`，算法查找目标序列 1 中第一个满足对于序列 2 中的某个`j`，`pred(i, j)`为`true`的元素 i。

如果`find_first_of`没有找到该子序列，则返回`ipt_end1`。如果`find_first_of`找到一个子序列，则返回一个指向第一个匹配子序列元素的`InputIterator`。（注意，如果`ipt_begin1`也是一个`ForwardIterator`，`find_first_of`将返回一个`ForwardIterator`。）

```
InputIterator find_first_of([ep], ipt_begin1, ipt_end1,
                            fwd_begin2, fwd_end2, [pred]);
```

##### 参数

+   一个可选的`std::execution`执行策略，`ep`（默认值：`std::execution::seq`）

+   一对`InputIterator`对象，`ipt_begin1` / `ipt_end1`，表示目标序列 1

+   一对`ForwardIterator`对象，`fwd_begin2` / `fwd_end2`，表示目标序列 2

+   一个可选的二元谓词`pred`，用于比较两个元素是否相等

##### 复杂度

**二次** 算法最多进行以下次数的比较或`pred`调用：

```
distance(ipt_begin1, ipt_end1) * distance(fwd_begin2, fwd_end2)
```

##### 示例

```
#include <algorithm>

TEST_CASE("find_first_of") {
  vector<string> words{ "Hen", "in", "a", "hat" }; ➊
  vector<string> indefinite_articles{ "a", "an" }; ➋
  const auto find_first_of_result = find_first_of(words.cbegin(),
                                                  words.cend(),
                                                  indefinite_articles.cbegin(),
                                                  indefinite_articles.cend()); ➌
  REQUIRE(*find_first_of_result == words[2]); ➍
}
```

在构造一个包含`string`对象的`vector`，名为`words` ➊，以及另一个名为`indefinite_articles` ➋之后，调用`find_first_of`来确定`words`中哪个元素开始的子序列等于`indefinite_articles` ➌。结果是`find_first_of_result`，其值为元素`a` ➍。

#### *adjacent_find*

`adjacent_find`算法找到序列中的第一个重复元素。

算法查找目标序列中第一个相邻元素相等的位置，或者如果提供了`pred`，算法查找目标序列中第一个满足`pred(i, i+1)`为`true`的元素。

如果`adjacent_find`没有找到该元素，则返回`fwd_end`。如果`adjacent_find`找到了该元素，则返回一个指向该元素的`ForwardIterator`。

```
ForwardIterator adjacent_find([ep], fwd_begin, fwd_end, [pred]);
```

##### 参数

+   一个可选的`std::execution`执行策略，`ep`（默认值：`std::execution::seq`）

+   一对`ForwardIterator`对象，`fwd_begin` / `fwd_end`，表示目标序列

+   一个可选的二元谓词`pred`用于比较两个元素是否相等

##### 复杂度

**线性** 如果没有提供执行策略，算法最多进行以下次数的比较或`pred`调用：

```
 min(distance(fwd_begin, i)+1, distance(fwd_begin, fwd_end)-1)
```

其中 i 是返回值的索引。

##### 示例

```
#include <algorithm>
TEST_CASE("adjacent_find") {
  vector<string> words{ "Icabod", "is", "itchy" }; ➊
  const auto first_letters_match = [](const auto& word1, const auto& word2) { ➋
    if (word1.empty() || word2.empty()) return false;
    return word1.front() == word2.front();
  };
  const auto adjacent_find_result = adjacent_find(words.cbegin(), words.cend(),
                                                  first_letters_match); ➌
  REQUIRE(*adjacent_find_result == words[1]); ➍
}
```

在构造一个包含`string`对象的`vector`，名为`words` ➊之后，构造一个名为`first_letters_match`的 lambda，该 lambda 接受两个单词并判断它们是否以相同的字母开头 ➋。调用`adjacent_find`来确定哪个元素与后续字母具有相同的首字母 ➌。结果`adjacent_find_result` ➍为`is`，因为它与`itchy`共享首字母 ➍。

#### *count*

`count`算法统计序列中符合某些用户定义标准的元素数量。

算法返回目标序列中`i`元素的数量，其中`pred`(`i`)为`true`，或者`value == i`。通常，`DifferenceType`是`size_t`，但它取决于`InputIterator`的实现。当你想要统计某个特定值的出现次数时，你使用`count`，而当你有一个更复杂的谓词想要用于比较时，你使用`count_if`。

```
DifferenceType count([ep], ipt_begin, ipt_end, value);
DifferenceType count_if([ep], ipt_begin, ipt_end, pred);
```

##### 参数

+   一个可选的`std::execution`执行策略，`ep`（默认值：`std::execution::seq`）

+   一对`InputIterator`对象，`ipt_begin` / `ipt_end`，表示目标序列。

+   一个`value`或一个一元谓词`pred`，用于评估目标序列中的元素`x`是否应被计数。

##### 复杂度

**线性** 如果没有给定执行策略，算法会进行`distance (ipt_begin, ipt_end)`次比较或`pred`调用。

##### 示例

```
#include <algorithm>
TEST_CASE("count") {
  vector<string> words{ "jelly", "jar", "and", "jam" }; ➊
  const auto n_ands = count(words.cbegin(), words.cend(), "and"); ➋
  REQUIRE(n_ands == 1); ➌

  const auto contains_a = [](const auto& word) { ➍
    return word.find('a') != string::npos;
  };
  const auto count_if_result = count_if(words.cbegin(), words.cend(),
                                        contains_a); ➎
  REQUIRE(count_if_result == 3); ➏
}
```

在构造一个包含`string`对象的`vector`，名为`words` ➊之后，你用它来调用`count`，值为`and` ➋。这会返回`1`，因为一个元素等于`and` ➌。接下来，你构造一个名为`contains_a`的 lambda，它接受一个单词并判断它是否包含`a` ➍。你调用`count_if`来确定有多少个单词包含`a` ➎。结果为`3`，因为有三个元素包含`a` ➏。

#### *不匹配*

`mismatch`算法用于查找两个序列中的第一个不匹配项。

算法找到来自序列 1 和序列 2 的第一个不匹配元素对`i`、`j`。具体来说，它找出第一个索引 n，使得`i = (ipt_begin1 + n)`；`j = (ipt_begin2 + n)`；并且`i != j`或`pred(i, j) == false`。

返回的`pair`中的迭代器类型与`ipt_begin1`和`ipt_begin2`的类型相等。

```
pair<Itr, Itr> mismatch([ep], ipt_begin1, ipt_end1,
                        ipt_begin2, [ipt_end2], [pred]);
```

##### 参数

+   一个可选的`std::execution`执行策略，`ep`（默认值：`std::execution::seq`）。

+   两对`InputIterator`，`ipt_begin1` / `ipt_end1`和`ipt_begin2` / `ipt_end2`，表示目标序列`1`和`2`。如果你没有提供`ipt_end2`，则序列 1 的长度隐含着序列 2 的长度。

+   一个可选的二元谓词`pred`用于比较两个元素是否相等。

##### 复杂度

**线性** 如果没有给定执行策略，最坏情况下算法会进行以下次数的比较或`pred`调用：

```
min(distance(ipt_begin1, ipt_end1), distance(ipt_begin2, ipt_end2))
```

##### 示例

```
#include <algorithm>

TEST_CASE("mismatch") {
  vector<string> words1{ "Kitten", "Kangaroo", "Kick" }; ➊
  vector<string> words2{ "Kitten", "bandicoot", "roundhouse" }; ➋
  const auto mismatch_result1 = mismatch(words1.cbegin(), words1.cend(),
                                         words2.cbegin()); ➌
  REQUIRE(*mismatch_result1.first == "Kangaroo"); ➍

  REQUIRE(*mismatch_result1.second == "bandicoot"); ➎
  const auto second_letter_matches = [](const auto& word1,
                                        const auto& word2) { ➏
    if (word1.size() < 2) return false;
    if (word2.size() < 2) return false;
    return word1[1] == word2[1];
  };
  const auto mismatch_result2 = mismatch(words1.cbegin(), words1.cend(),
                                     words2.cbegin(), second_letter_matches); ➐
  REQUIRE(*mismatch_result2.first == "Kick"); ➑
  REQUIRE(*mismatch_result2.second == "roundhouse"); ➒
}
```

在构造两个`vector`类型的`string`序列，名为`words1` ➊和`words2` ➋之后，你将它们作为`mismatch`的目标序列 ➌。这会返回一个`pair`，指向元素`Kangaroo`和`bandicoot` ➍ ➎。接下来，你构造一个名为`second_letter_matches`的 lambda，它接受两个单词并判断它们的第二个字母是否相同 ➏。你调用`mismatch`来找出第一个第二个字母不匹配的元素对 ➐。结果是元素对`Kick` ➑和`roundhouse` ➒。

#### *相等*

`equal`算法用于判断两个序列是否相等。

算法用于判断序列 1 的元素是否与序列 2 的元素相等。

```
bool equal([ep], ipt_begin1, ipt_end1, ipt_begin2, [ipt_end2], [pred]);
```

##### 参数

+   一个可选的`std::execution`执行策略`ep`（默认：`std::execution::seq`）。

+   两对`InputIterator`，`ipt_begin1` / `ipt_end1` 和 `ipt_begin2` / `ipt_end2`，表示目标序列 1 和 2。如果没有提供`ipt_end2`，则序列 1 的长度意味着序列 2 的长度。

+   一个可选的二元谓词`pred`，用于比较两个元素是否相等。

##### 复杂度

**线性** 当没有给出执行策略时，算法在最坏情况下进行以下数量的比较或调用`pred`：

```
min(distance(ipt_begin1, ipt_end1), distance(ipt_begin2, ipt_end2))
```

##### 示例

```
#include <algorithm>

TEST_CASE("equal") {
  vector<string> words1{ "Lazy", "lion", "licks" }; ➊
  vector<string> words2{ "Lazy", "lion", "kicks" }; ➋
 const auto equal_result1 = equal(words1.cbegin(), words1.cend(),
                                    words2.cbegin()); ➌
  REQUIRE_FALSE(equal_result1); ➍

  words2[2] = words1[2]; ➎
  const auto equal_result2 = equal(words1.cbegin(), words1.cend(),
                                    words2.cbegin()); ➏
  REQUIRE(equal_result2); ➐
}
```

在构造两个名为`words1`和`words2`的`vector<string>` ➊ ➋ 后，您将它们作为`equal`的目标序列 ➌。因为它们的最后一个元素`lick`和`kick`不相等，`equal_result1`为`false` ➍。在将`words2`的第三个元素设置为`words1`的第三个元素 ➎ 后，您再次使用相同的参数调用`equal` ➏。因为序列现在相同，`equal_result2`为`true` ➐。

#### *is_permutation*

`is_permutation`算法确定两个序列是否是排列，即它们包含相同的元素，但可能顺序不同。

算法确定是否存在序列 2 的某个排列，使得序列 1 的元素等于该排列的元素。

```
bool is_permutation([ep], fwd_begin1, fwd_end1, fwd_begin2, [fwd_end2], [pred]);
```

##### 参数

+   一个可选的`std::execution`执行策略`ep`（默认：`std::execution::seq`）。

+   两对`ForwardIterator`，`fwd_begin1` / `fwd_end1` 和 `fwd_begin2` / `fwd_end2`，表示目标序列 1 和 2。如果没有提供`fwd_end2`，则序列 1 的长度意味着序列 2 的长度。

+   一个可选的二元谓词`pred`，用于比较两个元素是否相等。

##### 复杂度

**二次方** 当没有给出执行策略时，算法在最坏情况下进行以下数量的比较或调用`pred`：

```
distance(fwd_begin1, fwd_end1) * distance(fwd_begin2, fwd_end2)
```

##### 示例

```
#include <algorithm>

TEST_CASE("is_permutation") {
  vector<string> words1{ "moonlight", "mighty", "nice" }; ➊
 vector<string> words2{ "nice", "moonlight", "mighty" }; ➋
  const auto result = is_permutation(words1.cbegin(), words1.cend(),
                                     words2.cbegin()); ➌
  REQUIRE(result); ➍
}
```

在构造两个名为`words1`和`words2`的`vector<string>` ➊ ➋ 后，您将它们作为`is_permutation`的目标序列 ➌。因为`words2`是`words1`的排列，`is_permutation`返回`true` ➍。

**注意**

*<algorithm>头文件还包含 next_permutation 和 prev_permutation，用于操作元素范围，以便生成排列。参见[alg.permutation.generators]。*

#### *search*

`search`算法用于定位子序列。

算法在序列 1 中定位序列 2。换句话说，它返回序列 1 中的第一个迭代器 i，使得对于每个非负整数`n`，`*(i + n)`等于`*(ipt_begin2 + n)`，或者如果提供了谓词`pred(*(i + n), *(ipt_begin2 + n))`为`true`。如果序列 2 为空，`search`算法返回`ipt_begin1`，如果没有找到子序列，则返回`ipt_begin2`。这与`find`不同，因为它定位的是子序列，而不是单个元素。

```
ForwardIterator search([ep], fwd_begin1, fwd_end1,
                             fwd_begin2, fwd_end2, [pred]);
```

##### 参数

+   一个可选的`std::execution`执行策略`ep`（默认：`std::execution::seq`）。

+   两对 `ForwardIterator`，`fwd_begin1` / `fwd_end1` 和 `fwd_begin2` / `fwd_end2`，表示目标序列 1 和 2

+   一个可选的二元谓词 `pred`，用于比较两个元素是否相等

##### 复杂度

**二次复杂度** 如果没有给定执行策略，最坏情况下该算法会进行以下次数的比较或 `pred` 调用：

```
distance(fwd_begin1, fwd_end1) * distance(fwd_begin2, fwd_end2)
```

##### 示例

```
#include <algorithm>

TEST_CASE("search") {
 vector<string> words1{ "Nine", "new", "neckties", "and",
                         "a", "nightshirt" }; ➊
  vector<string> words2{ "and", "a", "nightshirt" }; ➋
  const auto search_result_1 = search(words1.cbegin(), words1.cend(),
                                      words2.cbegin(), words2.cend()); ➌
  REQUIRE(*search_result_1 == "and"); ➍

  vector<string> words3{ "and", "a", "nightpant" }; ➎
  const auto search_result_2 = search(words1.cbegin(), words1.cend(),
                                      words3.cbegin(), words3.cend()); ➏
  REQUIRE(search_result_2 == words1.cend()); ➐
}
```

在构建了两个名为 `words1` ➊ 和 `words2` ➋ 的 `vector` 类型的 `string` 序列后，你将它们作为 `search` 的目标序列 ➌。由于 `words2` 是 `words1` 的子序列，`search` 返回指向 `and` 的迭代器 ➍。包含 `string` 对象的 `vector` `words3` ➎ 包含了单词 `nightpant` 而不是 `nightshirt`，因此使用它而不是 `words2` 调用 `search` 会返回 `words1` 的末尾迭代器 ➐。

#### *search_n*

`search_n` 算法定位包含相同连续值的子序列。

该算法在序列中查找 `count` 个连续的 `values`，并返回一个指向第一个 `value` 的迭代器，或者如果未找到此子序列，则返回 `fwd_end`。与 `adjacent_find` 不同，它定位的是一个子序列而不是单个元素。

```
ForwardIterator search_n([ep], fwd_begin, fwd_end, count, value, [pred]);
```

##### 参数

+   一个可选的 `std::execution` 执行策略，`ep`（默认值：`std::execution::seq`）

+   一对 `ForwardIterator`，`fwd_begin` / `fwd_end`，表示目标序列

+   一个整数型 `count` 值，表示你想查找的连续匹配的数量

+   一个 `value`，表示你要查找的元素

+   一个可选的二元谓词 `pred`，用于比较两个元素是否相等

##### 复杂度

**线性** 如果没有给定执行策略，最坏情况下该算法会进行 `distance(fwd_begin, fwd_end)` 次比较或 `pred` 调用。

##### 示例

```
#include <algorithm>

TEST_CASE("search_n") {
  vector<string> words{ "an", "orange", "owl", "owl", "owl", "today" }; ➊
  const auto result = search_n(words.cbegin(), words.cend(), 3, "owl"); ➋
  REQUIRE(result == words.cbegin() + 2); ➌
}
```

在构建了一个名为 `words` 的 `vector` 类型的 `string` 序列后 ➊，你将它作为 `search_n` 的目标序列 ➋。由于 `words` 中包含三个 `owl` 单词的实例，它会返回指向第一个实例的迭代器 ➌。

### 变异序列操作

一个 *变异序列操作* 是一种算法，它对序列进行计算，并允许以某种方式修改序列。本节中解释的每个算法都位于 `<algorithm>` 头文件中。

#### *copy*

`copy` 算法将一个序列复制到另一个序列中。

该算法将目标序列复制到 `result` 中，并返回接收序列的末尾迭代器。你有责任确保 `result` 表示一个具有足够空间来存储目标序列的序列。

```
OutputIterator copy([ep], ipt_begin, ipt_end, result);
```

##### 参数

+   一个可选的 `std::execution` 执行策略，`ep`（默认值：`std::execution::seq`）

+   一对 `InputIterator` 对象，`ipt_begin` 和 `ipt_end`，表示目标序列

+   一个 `OutputIterator`，`result`，接收复制的序列

##### 复杂度

**线性** 该算法会从目标序列中复制元素，恰好执行 `distance(ipt_begin, ipt_end)` 次。

##### 附加要求

序列 1 和 2 必须不重叠，除非操作是 *向左复制*。例如，对于一个包含 10 个元素的向量 `v`，`std::copy(v.begin()+3, v.end(), v.begin())` 是合法的，但 `std::copy(v.begin(), v.begin()+7, v.begin()+3)` 不是。

**注意**

*回顾一下“插入迭代器”中的 back_inserter，见 第 464 页，它返回一个输出迭代器，将写操作转换为在底层容器上的插入操作。*

##### 示例

```
#include <algorithm>

TEST_CASE("copy") {
  vector<string> words1{ "and", "prosper" }; ➊
  vector<string> words2{ "Live", "long" }; ➋
  copy(words1.cbegin(), words1.cend(), ➌
       back_inserter(words2)➍);
  REQUIRE(words2 == vector<string>{ "Live", "long", "and", "prosper" }); ➎
}
```

在构造两个 `vector` 类型的 `string` 对象后 ➊ ➋，你使用 `copy`，将 `words1` 作为待复制序列 ➌，`words2` 作为目标序列 ➍。结果是 `words2` 包含了 `words1` 的内容，并追加到原始内容后 ➎。

#### *copy_n*

`copy_n` 算法将一个序列复制到另一个序列中。

该算法将目标序列复制到 `result` 中，并返回接收序列的末尾迭代器。你需要确保 `result` 代表一个具有足够空间存储目标序列的序列，并且 `n` 代表目标序列的正确长度。

```
OutputIterator copy_n([ep], ipt_begin, n, result);
```

##### 参数

+   一个可选的 `std::execution` 执行策略，`ep`（默认值：`std::execution::seq`）

+   一个表示目标序列起始位置的开始迭代器，`ipt_begin`

+   目标序列的大小，`n`

+   一个 `OutputIterator result`，接收复制后的序列

##### 复杂度

**线性** 该算法将从目标序列中复制 `distance(ipt_begin, ipt_end)` 次元素。

##### 附加要求

序列 1 和 2 必须不包含相同的对象，除非操作是 *向左复制*。

##### 示例

```
#include <algorithm>

TEST_CASE("copy_n") {
  vector<string> words1{ "on", "the", "wind" }; ➊
  vector<string> words2{ "I'm", "a", "leaf" }; ➋
  copy_n(words1.cbegin(), words1.size(), ➌
         back_inserter(words2)); ➍
  REQUIRE(words2 == vector<string>{ "I'm", "a", "leaf",
                                    "on", "the", "wind" }); ➎
}
```

在构造两个 `vector` 类型的 `string` 对象后 ➊ ➋，你使用 `copy_n`，将 `words1` 作为待复制序列 ➌，`words2` 作为目标序列 ➍。结果是 `words2` 包含了 `words1` 的内容，并追加到原始内容后 ➎。

#### *copy_backward*

`copy_backward` 算法将一个序列的元素反向复制到另一个序列中。

该算法将序列 1 复制到序列 2 中，并返回接收序列的末尾迭代器。元素会反向复制，但在目标序列中仍然按原顺序出现。你需要确保序列 1 有足够的空间来存储序列 2。

```
OutputIterator copy_backward([ep], ipt_begin1, ipt_end1, ipt_end2);
```

##### 参数

+   一个可选的 `std::execution` 执行策略，`ep`（默认值：`std::execution::seq`）

+   一对 `InputIterator` 对象，`ipt_begin1` 和 `ipt_end1`，表示序列 1

+   一个 `InputIterator`，`ipt_end2`，表示序列 2 末尾之后的位置

##### 复杂度

**线性** 该算法将从目标序列中复制 `distance(ipt_begin1, ipt_end1)` 次元素。

##### 附加要求

序列 1 和 2 必须不重叠。

##### 示例

```
#include <algorithm>

TEST_CASE("copy_backward") {
 vector<string> words1{ "A", "man", "a", "plan", "a", "bran", "muffin" }; ➊
  vector<string> words2{ "a", "canal", "Panama" }; ➋
  const auto result = copy_backward(words2.cbegin(), words2.cend(), ➌
                                    words1.end()); ➍
  REQUIRE(words1 == vector<string>{ "A", "man", "a", "plan",
                                    "a", "canal", "Panama" }); ➎
}
```

在构造了两个`string`类型的`vector`对象 ➊ ➋后，你调用`copy_backward`，以`words2`作为要复制的序列 ➌，`words1`作为目标序列 ➍。结果是，`word2`的内容替换了`words1`的最后三个单词 ➎。

#### *move*

`move`算法将一个序列移动到另一个序列中。

算法将目标序列移动并返回接收序列的结束迭代器。你有责任确保目标序列的元素至少与源序列一样多。

```
OutputIterator move([ep], ipt_begin, ipt_end, result);
```

##### 参数

+   一个可选的`std::execution`执行策略，`ep`（默认值：`std::execution::seq`）

+   一对`InputIterator`对象，`ipt_begin`和`ipt_end`，表示目标序列

+   一个`InputIterator`，`result`，表示要移动到的序列的起始位置

##### 复杂度

**线性** 算法从目标序列中移动元素，恰好`distance(ipt_begin, ipt_end)`次。

##### 附加要求

+   序列不得重叠，除非是*向左移动*。

+   类型必须是可移动的，但不一定是可复制的。

##### 示例

```
#include <algorithm>

struct MoveDetector { ➊
  MoveDetector() : owner{ true } {} ➋
  MoveDetector(const MoveDetector&) = delete;
  MoveDetector& operator=(const MoveDetector&) = delete;
  MoveDetector(MoveDetector&& o) = delete;
  MoveDetector& operator=(MoveDetector&&) { ➌
 o.owner = false;
    owner = true;
    return *this;
  }
  bool owner;
};

TEST_CASE("move") {
  vector<MoveDetector> detectors1(2); ➍
  vector<MoveDetector> detectors2(2); ➎
  move(detectors1.begin(), detectors1.end(), detectors2.begin()); ➏
  REQUIRE_FALSE(detectors1[0].owner); ➐
  REQUIRE_FALSE(detectors1[1].owner); ➑
  REQUIRE(detectors2[0].owner); ➒
  REQUIRE(detectors2[1].owner); ➓
}
```

首先，你声明了`MoveDetector`类 ➊，它定义了一个默认构造函数，将唯一的成员`owner`设置为`true` ➋。它删除了复制构造函数和移动构造函数，以及复制赋值运算符，但定义了一个移动赋值运算符，用于交换`owner` ➌。

在构造了两个`MoveDetector`对象的`vector` ➍ ➎后，你调用`move`，以`detectors1`作为要`move`的序列，`detectors2`作为目标序列 ➏。结果是，`detector1`的元素处于*moved from*状态 ➐➑，而`detectors2`的元素被移动到`detectors2` ➒➓。

#### *move_backward*

`move_backward`算法将一个序列的反向内容移动到另一个序列中。

算法将序列 1 移动到序列 2，并返回一个指向最后一个移动元素的迭代器。元素向后移动，但会以原始顺序出现在目标序列中。你有责任确保目标序列的元素至少与源序列一样多。

```
OutputIterator move_backward([ep], ipt_begin, ipt_end, result);
```

##### 参数

+   一个可选的`std::execution`执行策略，`ep`（默认值：`std::execution::seq`）

+   一对`InputIterator`对象，`ipt_begin`和`ipt_end`，表示目标序列

+   一个`InputIterator`，`result`，表示要移动到的序列

##### 复杂度

**线性** 算法从目标序列中移动元素，恰好`distance(ipt_begin, ipt_end)`次。

##### 附加要求

+   序列不得重叠。

+   类型必须是可移动的，但不一定是可复制的。

##### 示例

```
#include <algorithm>

struct MoveDetector { ➊
--snip--
};

TEST_CASE("move_backward") {
  vector<MoveDetector> detectors1(2); ➋
  vector<MoveDetector> detectors2(2); ➌
  move_backward(detectors1.begin(), detectors1.end(), detectors2.end()); ➍
  REQUIRE_FALSE(detectors1[0].owner); ➎
  REQUIRE_FALSE(detectors1[1].owner); ➏
  REQUIRE(detectors2[0].owner); ➐
  REQUIRE(detectors2[1].owner); ➑
}
```

首先，你声明了`MoveDetector`类 ➊（有关实现，请参见“`move`”章节，第 595 页）。

在构造了两个`MoveDetector`对象的`vector`后 ➋ ➌，你调用`move`，将`detectors1`作为要`move`的序列，`detectors2`作为目标序列 ➍。结果是，`detector1`的元素处于*已移动出*状态 ➎➏，`detector2`的元素处于*已移动入*状态 ➐➑。

#### *swap_ranges*

`swap_ranges`算法将一个序列的元素交换到另一个序列中。

该算法对序列 1 和序列 2 的每个元素调用`swap`，并返回接收序列的结束迭代器。你有责任确保目标序列的元素数量至少与源序列相同。

```
OutputIterator swap_ranges([ep], ipt_begin1, ipt_end1, ipt_begin2);
```

##### 参数

+   一个可选的`std::execution`执行策略，`ep`（默认值：`std::execution::seq`）。

+   一对`ForwardIterator`，`ipt_begin1`和`ipt_end1`，表示序列 1。

+   一个`ForwardIterator`，`ipt_begin2`，表示序列 2 的开始。

##### 复杂度

**线性** 该算法会调用`swap`正好`distance(ipt_begin1, ipt_end1)`次。

##### 附加要求

每个序列中包含的元素必须是可交换的。

##### 示例

```
#include <algorithm>

TEST_CASE("swap_ranges") {
  vector<string> words1{ "The", "king", "is", "dead." }; ➊
  vector<string> words2{ "Long", "live", "the", "king." }; ➋
  swap_ranges(words1.begin(), words1.end(), words2.begin()); ➌
  REQUIRE(words1 == vector<string>{ "Long", "live", "the", "king." }); ➍
  REQUIRE(words2 == vector<string>{ "The", "king", "is", "dead." }); ➎
}
```

在构造了两个包含`string`对象的`vector`后 ➊ ➋，你调用`swap`，将`words1`和`words2`作为要交换的序列 ➌。结果是`words1`和`words2`交换内容 ➍ ➎。

#### *transform*

`transform`算法修改一个序列中的元素，并将其写入另一个序列。

该算法对目标序列的每个元素调用`unary_op`并将其输出到输出序列，或者对每个目标序列中的相应元素调用`binary_op`。

```
OutputIterator transform([ep], ipt_begin1, ipt_end1, result, unary_op);
OutputIterator transform([ep], ipt_begin1, ipt_end1, ipt_begin2,
                         result, binary_op);
```

##### 参数

+   一个可选的`std::execution`执行策略，`ep`（默认值：`std::execution::seq`）。

+   一对`InputIterator`对象，`ipt_begin1`和`ipt_end1`，表示目标序列。

+   一个可选的`InputIterator`，`ipt_begin2`，表示第二个目标序列。你必须确保第二个目标序列的元素数量至少与第一个目标序列相同。

+   一个`OutputIterator`，`result`，表示输出序列的开始。

+   一个一元操作，`unary_op`，用于将目标序列的元素转换为输出序列的元素。如果你提供了两个目标序列，则提供一个二元操作`binary_op`，它接受每个目标序列中的一个元素，并将它们转换为输出序列中的元素。

##### 复杂度

**线性** 该算法会调用`unary_op`或`binary_op`，正好调用`distance(ipt_begin1, ipt_end1)`次。

##### 示例

```
#include <algorithm>
#include <boost/algorithm/string/case_conv.hpp>

TEST_CASE("transform") {
  vector<string> words1{ "farewell", "hello", "farewell", "hello" }; ➊
  vector<string> result1;
  auto upper = [](string x) { ➋
    boost::algorithm::to_upper(x);
    return x;
  };
  transform(words1.begin(), words1.end(), back_inserter(result1), upper); ➌
  REQUIRE(result1 == vector<string>{ "FAREWELL", "HELLO",
                                     "FAREWELL", "HELLO" }); ➍

  vector<string> words2{ "light", "human", "bro", "quantum" }; ➎
  vector<string> words3{ "radar", "robot", "pony", "bit" }; ➏
  vector<string> result2;
  auto portmantize = [](const auto &x, const auto &y) { ➐
    const auto x_letters = min(size_t{ 2 }, x.size());
    string result{ x.begin(), x.begin() + x_letters };
    const auto y_letters = min(size_t{ 3 }, y.size());
    result.insert(result.end(), y.end() - y_letters, y.end() );
    return result;
  };
  transform(words2.begin(), words2.end(), words3.begin(),
            back_inserter(result2), portmantize); ➑
  REQUIRE(result2 == vector<string>{ "lidar", "hubot", "brony", "qubit" }); ➒
}
```

在构造了一个包含`string`对象的`vector`后 ➊，你构造了一个名为`upper`的 lambda，它按值接受一个`string`并使用 Boost 的`to_upper`算法将其转换为大写，如第十五章中讨论的 ➋。你使用`transform`，将`words1`作为目标序列，使用一个空的`results1``vector`的`back_inserter`，并将`upper`作为一元操作 ➌。调用`transform`后，`results1`包含了`words1`的大写版本 ➍。

在第二个示例中，您构造了两个 `vector` 类型的 `string` 对象 ➎➏。您还构造了一个名为 `portmantize` 的 lambda 函数，该函数接受两个 `string` 对象 ➐。该 lambda 返回一个新的 `string`，包含第一个参数的前两个字母和第二个参数的后三个字母。您将两个目标序列、一个指向空 `vector` 的 `back_inserter` 以及 `portmantize` ➑ 一同传递。`result2` 包含了 `words1` 和 `words2` 的混合词 ➒。

#### *replace*

`replace` 算法将序列中的某些元素替换为新的元素。

算法查找目标序列元素 x，对于满足 `x == old_ref` 或 `pred(x) == true` 的元素，将其赋值为 `new_ref`。

```
void replace([ep], fwd_begin, fwd_end, old_ref, new_ref);
void replace_if([ep], fwd_begin, fwd_end, pred, new_ref);
void replace_copy([ep], fwd_begin, fwd_end, result, old_ref, new_ref);
void replace_copy_if([ep], fwd_begin, fwd_end, result, pred, new_ref);
```

##### 参数

+   一个可选的 `std::execution` 执行策略 `ep`（默认值：`std::execution::seq`）

+   一对 `ForwardIterator`，`fwd_begin` 和 `fwd_end`，表示目标序列

+   一个 `OutputIterator`，`result`，表示输出序列的起始位置

+   一个 `old` `const` 引用，表示要查找的元素

+   一个一元谓词 `pred`，用于判断元素是否符合替换条件

+   一个 `new_ref` `const` 引用，表示要替换的元素

##### 复杂度

**线性** 算法调用 `pred` 恰好 `distance(fwd_begin, fwd_end)` 次。

##### 附加要求

每个序列中的元素必须能够与 `old_ref` 进行比较，并且能够赋值给 `new_ref`。

##### 示例

```
#include <algorithm>
#include <string_view>

TEST_CASE("replace") {
  using namespace std::literals; ➊
  vector<string> words1{ "There", "is", "no", "try" }; ➋
  replace(words1.begin(), words1.end(), "try"sv, "spoon"sv); ➌
  REQUIRE(words1 == vector<string>{ "There", "is", "no", "spoon" }); ➍

  const vector<string> words2{ "There", "is", "no", "spoon" }; ➎
 vector<string> words3{ "There", "is", "no", "spoon" }; ➏
  auto has_two_os = [](const auto& x) { ➐
    return count(x.begin(), x.end(), 'o') == 2;
  };
  replace_copy_if(words2.begin(), words2.end(), words3.begin(), ➑
                  has_two_os, "try"sv);
  REQUIRE(words3 == vector<string>{ "There", "is", "no", "try" }); ➒
}
```

首先引入 `std::literals` 命名空间 ➊，这样您就可以稍后使用 `string_view` 字面量。构造一个包含 `string` 对象的 `vector` ➋ 后，您调用 `replace` 并使用该 `vector` ➌ 来将所有 `try` 替换为 `spoon` ➍。

在第二个示例中，您构造了两个 `vector` 类型的 `string` 对象 ➎➏ 和一个名为 `has_two_os` 的 lambda 函数，该函数接受一个字符串并返回 `true`，如果该字符串恰好包含两个 `o` ➐。然后，您将 `words2` 作为目标序列，`words3` 作为目标序列传递给 `replace_copy_if`，它对 `words2` 中的每个元素应用 `has_two_os`，并将满足条件的元素替换为 `try` ➑。结果是 `words2` 不受影响，而 `words3` 中的元素 `spoon` 被替换为 `try` ➒。

#### *fill*

`fill` 算法用某个值填充序列。

算法将一个值写入目标序列的每个元素。`fill_n` 函数返回 `opt_begin + n`。

```
void fill([ep], fwd_begin, fwd_end, value);
OutputIterator fill_n([ep], opt_begin, n, value);
```

##### 参数

+   一个可选的 `std::execution` 执行策略 `ep`（默认值：`std::execution::seq`）

+   一个 `ForwardIterator`，`fwd_begin`，表示目标序列的起始位置

+   一个 `ForwardIterator`，`fwd_end`，表示序列末尾的下一个位置

+   一个表示元素数量的 `Size n`

+   一个要写入目标序列每个元素的 `value`

##### 复杂度

**线性** 算法将 `value` 赋值给目标序列的每个元素，恰好 `distance(fwd_begin, fwd_end)` 或 `n` 次。

##### 附加要求

+   `value` 参数必须能够写入序列。

+   `Size`类型的对象必须可以转换为整型。

##### 示例

```
#include <algorithm>

// If police police police police, who polices the police police?
TEST_CASE("fill") {
  vector<string> answer1(6); ➊
  fill(answer1.begin(), answer1.end(), "police"); ➋
  REQUIRE(answer1 == vector<string>{ "police", "police", "police",
                                     "police", "police", "police" }); ➌

  vector<string> answer2; ➍
  fill_n(back_inserter(answer2), 6, "police"); ➎
  REQUIRE(answer2 == vector<string>{ "police", "police", "police",
                                     "police", "police", "police" }); ➏
}
```

你首先初始化一个包含六个空元素的`vector`，其中包含`string`对象 ➊。接下来，使用`vector`作为目标序列并将`police`作为值来调用`fill` ➋。结果是你的`vector`包含六个`police` ➌。

在第二个示例中，你初始化一个空的`vector`，其中包含`string`对象 ➍。然后，你用`back_inserter`调用`fill_n`，指向空的`vector`、长度为 6，并将`police`作为值 ➎。结果和之前一样：你的`vector`包含六个`police` ➏。

#### *generate*

`generate`算法通过调用一个函数对象来填充序列。

算法调用`generator`并将结果赋值到目标序列中。`generate_n`函数返回`opt_begin+n`。

```
void generate([ep], fwd_begin, fwd_end, generator);
OutputIterator generate_n([ep], opt_begin, n, generator);
```

##### 参数

+   可选的`std::execution`执行策略，`ep`（默认值：`std::execution::seq`）

+   一个`ForwardIterator`，`fwd_begin`，表示目标序列的起始位置

+   一个`ForwardIterator`，`fwd_end`，表示序列末尾之后的位置

+   一个表示元素数量的`Size n`

+   一个`generator`，当没有参数调用时，生成一个元素以写入目标序列

##### 复杂度

**线性** 算法调用`generator`恰好`distance(fwd_begin, fwd_end)`次或`n`次。

##### 附加要求

+   `value`参数必须可以写入序列。

+   `Size`类型的对象必须可以转换为整型。

##### 示例

```
#include <algorithm>

TEST_CASE("generate") {
  auto i{ 1 }; ➊
  auto pow_of_2 = [&i]() { ➋
    const auto tmp = i;
    i *= 2;
    return tmp;
  };
  vector<int> series1(6); ➌
  generate(series1.begin(), series1.end(), pow_of_2); ➍
  REQUIRE(series1 == vector<int>{ 1, 2, 4, 8, 16, 32 }); ➎

  vector<int> series2; ➏
  generate_n(back_inserter(series2), 6, pow_of_2); ➐
  REQUIRE(series2 == vector<int>{ 64, 128, 256, 512, 1024, 2048 }); ➑
}
```

你首先初始化一个名为`i`的`int`为 1 ➊。接着，你创建一个名为`pow_of_2`的 lambda，它通过引用获取`i` ➋。每次调用`pow_of_2`时，它将`i`加倍，并返回加倍前的值。然后，你初始化一个包含六个元素的`vector`，其元素类型为`int` ➌。然后，你用`vector`作为目标序列，`pow_of_2`作为生成器来调用`generate` ➍。结果是`vector`包含前六个 2 的幂 ➎。

在第二个示例中，你初始化一个空的`vector`，其中包含`int`对象 ➏。接下来，你使用`back_inserter`调用`generate_n`，传入空的`vector`、大小为 6 和`pow_of_2`作为生成器 ➐。`result`是接下来的六个 2 的幂 ➑。注意，`pow_of_2`有状态，因为它通过引用捕获了`i`。

#### *remove*

`remove`算法从序列中移除某些元素。

算法将所有`pred`为`true`或元素等于`value`的元素移动，确保剩余元素的顺序保持不变，并返回指向第一个移动元素的迭代器。这个迭代器被称为结果序列的*逻辑结束*。序列的物理大小保持不变，通常`remove`调用后会跟着调用容器的`erase`方法。

```
ForwardIterator remove([ep], fwd_begin, fwd_end, value);
ForwardIterator remove_if([ep], fwd_begin, fwd_end, pred);
ForwardIterator remove_copy([ep], fwd_begin, fwd_end, result, value);
ForwardIterator remove_copy_if([ep], fwd_begin, fwd_end, result, pred);
```

##### 参数

+   可选的`std::execution`执行策略，`ep`（默认值：`std::execution::seq`）

+   一对`ForwardIterator`，`fwd_begin`和`fwd_end`，表示目标序列

+   一个`OutputIterator`，`result`，表示目标序列（如果是复制的情况下）

+   一个表示要移除元素的`value`

+   一个一元谓词`pred`，用于判断元素是否符合移除的标准

##### 复杂度

**线性** 该算法调用`pred`或与`value`进行比较的次数恰好是`distance(fwd_begin, fwd_end)`次。

##### 附加要求

+   目标序列的元素必须是可移动的。

+   如果进行复制，元素必须是可复制的，且目标序列和源序列不能重叠。

##### 示例

```
#include <algorithm>

TEST_CASE("remove") {
  auto is_vowel = [](char x) { ➊
    const static string vowels{ "aeiouAEIOU" };
    return vowels.find(x) != string::npos;
  };
  string pilgrim = "Among the things Billy Pilgrim could not change "
                   "were the past, the present, and the future."; ➋
  const auto new_end = remove_if(pilgrim.begin(), pilgrim.end(), is_vowel); ➌
  REQUIRE(pilgrim == "mng th thngs Blly Plgrm cld nt chng wr th pst, "
                     "th prsnt, nd th ftr.present, and the future."); ➍

  pilgrim.erase(new_end, pilgrim.end()); ➎
  REQUIRE(pilgrim == "mng th thngs Blly Plgrm cld nt chng wr th "
                     "pst, th prsnt, nd th ftr."); ➏
}
```

首先，你创建一个名为`is_vowel`的 lambda 函数，当给定的`char`是元音时返回`true` ➊。接着，构造一个名为`pilgrim`的`string`，其中包含一个句子 ➋。然后，调用`remove_if`，以`pilgrim`作为目标句子，`is_vowel`作为谓词 ➌。每当`remove_if`遇到一个元音时，它会将剩余字符向左移动，从而消除句子中的所有元音。结果是，`pilgrim`包含了原始句子，去除了元音，并加上了`present, and the future.`这一短语 ➍。这个短语包含 24 个字符，这正好是`remove_if`从原句中移除的元音数量。`present, and the future.`这个短语是移除过程中剩余字符串移动所产生的碎片。

为了消除这些剩余元素，你保存`remove_if`返回的迭代器`new_end`，它指向新目标序列中最后一个字符后的一个位置，即`present, and the future.`中的`p`。要消除这些元素，你只需在`pilgrim`上使用`erase`方法，`erase`方法有一个接受半开区间的重载。你将`remove_if`返回的逻辑末尾`new_end`作为开始迭代器，同时将`pilgrim.end()`作为结束迭代器 ➎。结果是，`pilgrim`现在等于去除元音后的原始句子 ➏。

这种将`remove`（或`remove_if`）与`erase`方法结合使用的方式，称为*擦除-移除惯用法*，被广泛应用。

#### *unique*

`unique`算法从序列中移除冗余元素。

该算法移动所有`pred`判断为`true`的重复元素，或是相等的元素，确保剩余的元素是唯一的且保留原始顺序。它返回指向新逻辑末尾的迭代器。与`std::remove`一样，物理存储不会改变。

```
ForwardIterator unique([ep], fwd_begin, fwd_end, [pred]);
ForwardIterator unique_copy([ep], fwd_begin, fwd_end, result, [pred]);
```

##### 参数

+   一个可选的`std::execution`执行策略，`ep`（默认值：`std::execution::seq`）

+   一对`ForwardIterator`，`fwd_begin`和`fwd_end`，表示目标序列

+   一个`OutputIterator`，`result`，表示目标序列（如果是复制的情况下）

+   一个二元谓词`pred`，用于判断两个元素是否相等

##### 复杂度

**线性** 该算法调用`pred`的次数恰好是`distance(fwd_begin, fwd_end) - 1`次。

##### 附加要求

+   目标序列的元素必须是可移动的。

+   如果是复制，目标序列的元素必须是可复制的，并且目标范围与目标位置的范围不能重叠。

##### 示例

```
#include <algorithm>

TEST_CASE("unique") {
  string without_walls = "Wallless"; ➊
  const auto new_end = unique(without_walls.begin(), without_walls.end()); ➋
  without_walls.erase(new_end, without_walls.end()); ➌
  REQUIRE(without_walls == "Wales"); ➍
}
```

你首先构造一个包含多个重复字符的`string` ➊。然后，你使用`string`作为目标序列调用`unique` ➋。这将返回逻辑上的结束位置，并将其赋值给`new_end`。接下来，你删除从`new_end`到`without_walls.end()`的范围 ➌。这是删除-移除模式的推论：最终你会得到`Wales`，其中包含连续的唯一字符 ➍。

#### *reverse*

`reverse`算法反转序列的顺序。

该算法通过交换元素或将其复制到目标序列来反转序列。

```
void reverse([ep], bi_begin, bi_end);
OutputIterator reverse_copy([ep], bi_begin, bi_end, result);
```

##### 参数

+   一个可选的`std::execution`执行策略，`ep`（默认值：`std::execution::seq`）

+   一对`BidirectionalIterator`，`bi_begin`和`bi_end`，表示目标序列。

+   一个`OutputIterator`，`result`，表示目标序列（如果是复制）。

##### 复杂度

**线性** 该算法精确调用`swap` `distance(bi_begin, bi_end)/2`次。

##### 附加要求

+   目标序列的元素必须是可交换的。

+   如果是复制，目标序列的元素必须是可复制的，并且目标范围与目标位置的范围不能重叠。

##### 示例

```
#include <algorithm>

TEST_CASE("reverse") {
 string stinky = "diaper"; ➊
  reverse(stinky.begin(), stinky.end()); ➋
  REQUIRE(stinky == "repaid"); ➌
}
```

你首先构造一个包含单词`diaper`的`string` ➊。接下来，你使用此`string`作为目标序列调用 reverse ➋。结果是单词`repaid` ➌。

#### *sample*

`sample`算法生成随机且稳定的子序列。

该算法从种群序列中抽取`min(pop_end - pop_begin, n)`个元素。稍微不直观的是，当且仅当`ipt_begin`是正向迭代器时，抽样结果才会被排序。它返回结果目标序列的结束位置。

```
OutputIterator sample([ep], ipt_begin, ipt_end, result, n, urb_generator);
```

##### 参数

+   一个可选的`std::execution`执行策略，`ep`（默认值：`std::execution::seq`）

+   一对`InputIterator`对象，`ipt_begin`和`ipt_end`，表示种群序列（即要抽样的序列）。

+   一个`OutputIterator`，`result`，表示目标序列。

+   一个`Distance`类型的`n`，表示要抽样的元素数量。

+   一个`UniformRandomBitGenerator`类型的`urb_generator`，例如在第十二章中介绍的 Mersenne Twister `std::mt19937_64`。

##### 复杂度

**线性** 该算法的复杂度与`distance(ipt_begin, ipt_end)`成比例。

##### 示例

```
#include <algorithm>
#include <map>
#include <string>
#include <iostream>
#include <iomanip>
#include <random>

using namespace std;

const string population = "ABCD"; ➊
const size_t n_samples{ 1'000'000 }; ➋
mt19937_64 urbg; ➌

void sample_length(size_t n) { ➍
  cout << "-- Length " << n << " --\n";
 map<string, size_t> counts; ➎
  for (size_t i{}; i < n_samples; i++) {
    string result;
    sample(population.begin(), population.end(),
           back_inserter(result), n, urbg); ➏
    counts[result]++;
  }
  for (const auto[sample, n] : counts) { ➐
    const auto percentage = 100 * n / static_cast<double>(n_samples);
    cout << percentage << " '" << sample << "'\n"; ➑
  }
}

int main() {
  cout << fixed << setprecision(1); ➒
  sample_length(0); ➓
  sample_length(1);
  sample_length(2);
  sample_length(3);
  sample_length(4);
}
-----------------------------------------------------------------------
-- Length 0 --
100.0 ''
-- Length 1 --
25.1 'A'
25.0 'B'
25.0 'C'
24.9 'D'
-- Length 2 --
16.7 'AB'
16.7 'AC'
16.6 'AD'
16.6 'BC'
16.7 'BD'
16.7 'CD'
-- Length 3 --
25.0 'ABC'
25.0 'ABD'
25.0 'ACD'
25.0 'BCD'
-- Length 4 --
100.0 'ABCD'
```

你首先构造一个名为`population`的`const string`，其中包含字母`ABCD` ➊。然后你初始化一个名为`n_samples`的`const size_t`，值为一百万 ➋，以及一个名为`urbg`的 Mersenne Twister ➌。所有这些对象的存储持续时间都是静态的。

此外，您初始化了一个名为`sample_length`的函数，该函数接受一个名为`n`的`size_t`参数➍。在该函数中，您构造一个`map`类型的`string`到`size_t`对象的集合➎，用于统计每次调用`sample`的频率。在一个`for`循环中，您调用`sample`，将`population`作为种群序列，将`back_inserter`作为目标序列的`result`字符串，`n`作为样本长度，以及`urbg`作为随机位生成器➏。

在一百万次迭代后，您迭代`counts`中的每个元素➐，并打印给定长度`n`的每个样本的概率分布➑。

在`main`函数中，您使用`fixed`和`setprecision`配置浮点数格式➒。最后，您使用从`0`到`4`的每个值调用`sample_length`➓。

因为`string`提供了随机访问迭代器，`sample`提供*稳定*（已排序）的样本。

**警告**

*请注意，输出不包含像 DC 或 CAB 这样的未排序样本。这个排序行为可能并不是算法名称中显而易见的，所以请小心！*

#### *洗牌*

`shuffle`算法生成随机排列。

该算法随机化目标序列，使得这些元素的每种可能排列出现的概率相等。

```
void shuffle(rnd_begin, rnd_end, urb_generator);
```

##### 参数

+   一对`RandomAccessIterator`（随机访问迭代器）`rnd_begin`和`rnd_end`，表示目标序列。

+   一个`UniformRandomBitGenerator`（均匀随机位生成器）`urb_generator`，例如在第十二章中介绍的梅森旋转算法`std::mt19937_64`

##### 复杂度

**线性** 该算法恰好交换`distance(rnd_begin, rnd_end) - 1`次。

##### 附加要求

目标序列的元素必须是可交换的。

##### 示例

```
#include <algorithm>
#include <map>
#include <string>
#include <iostream>
#include <random>
#include <iomanip>

using namespace std;

int main() {
  const string population = "ABCD"; ➊
  const size_t n_samples{ 1'000'000 }; ➋
  mt19937_64 urbg; ➌
 map<string, size_t> samples; ➍
  cout << fixed << setprecision(1); ➎
  for (size_t i{}; i < n_samples; i++) {
    string result{ population }; ➏
    shuffle(result.begin(), result.end(), urbg); ➐
    samples[result]++; ➑
  }
  for (const auto[sample, n] : samples) { ➒
    const auto percentage = 100 * n / static_cast<double>(n_samples);
    cout << percentage << " '" << sample << "'\n"; ➓
  }
}
-----------------------------------------------------------------------
4.2 'ABCD'
4.2 'ABDC'
4.1 'ACBD'
4.2 'ACDB'
4.2 'ADBC'
4.2 'ADCB'
4.2 'BACD'
4.2 'BADC'
4.1 'BCAD'
4.2 'BCDA'
4.1 'BDAC'
4.2 'BDCA'
4.2 'CABD'
4.2 'CADB'
4.1 'CBAD'
4.1 'CBDA'
4.2 'CDAB'
4.1 'CDBA'
4.2 'DABC'
4.2 'DACB'
4.2 'DBAC'
4.1 'DBCA'
4.2 'DCAB'
4.2 'DCBA'
```

您首先构造一个名为`population`的`const string`，其中包含字母`ABCD`➊。您还初始化一个名为`n_samples`的`const size_t`，它的值为一百万➋，一个名为`urbg`的梅森旋转算法（Mersenne Twister）➌，以及一个`map`类型的`string`到`size_t`对象的集合➍，用于统计每个`shuffle`样本的频率。此外，您使用`fixed`和`setprecision`配置浮点数格式➎。

在`for`循环中，您将`population`复制到一个名为`sample`的新字符串中，因为`shuffle`会修改目标序列➏。然后，您调用`shuffle`，将`result`作为目标序列，`urbg`作为随机位生成器➐，并将结果记录在`samples`中➑。

最后，您迭代`sample`中的每个元素➒并打印每个样本的概率分布➓。

请注意，与`sample`不同，`shuffle`始终生成一个*无序*的元素分布。

### 排序及相关操作

*排序操作*是一个将序列重新排列为所需方式的算法。

每个排序算法都有两个版本：一个接受名为 *比较操作符* 的函数对象，另一个使用 `operator<`。比较操作符是一个函数对象，可以使用两个对象进行比较。它返回 `true` 如果第一个参数是 *小于* 第二个参数；否则返回 `false`。`x < y` 的排序解释是 `x` 排在 `y` 前面。本节中解释的所有算法都位于 `<algorithm>` 头文件中。

**注意**

*注意，operator< 是一个有效的比较操作符。*

比较操作符必须是传递的。这意味着对于任何元素 `a`、`b` 和 `c`，比较操作符 `comp` 必须保持以下关系：如果 `comp(a, b)` 和 `comp(b, c)`，那么 `comp(a, c)`。这应该是合理的：如果 `a` 排在 `b` 前面，且 `b` 排在 `c` 前面，那么 `a` 必须排在 `c` 前面。

#### *sort*

`sort` 算法对序列进行排序（不稳定）。

**注意**

*稳定排序会保留相等元素的相对顺序，而不稳定排序可能会重新排序它们。*

算法就地对目标序列进行排序。

```
void sort([ep], rnd_begin, rnd_end, [comp]);
```

##### 参数

+   一个可选的 `std::execution` 执行策略，`ep`（默认值：`std::execution::seq`）

+   一对 `RandomAccessIterator`，`rnd_begin` 和 `rnd_end`，表示目标序列

+   一个可选的比较操作符，`comp`

##### 复杂度

**准线性** O(N log N)，其中 N = `distance(rnd_begin, rnd_end)`

##### 附加要求

目标序列的元素必须是可交换的、可移动构造的和可移动赋值的。

##### 示例

```
#include <algorithm>

TEST_CASE("sort") {
  string goat_grass{ "spoilage" }; ➊
  sort(goat_grass.begin(), goat_grass.end()); ➋
  REQUIRE(goat_grass == "aegilops"); ➌
}
```

你首先构造一个包含单词 `spoilage` 的 `string` ➊。接着，你用这个 `string` 作为目标序列调用 `sort` ➋。结果是 `goat_``grass` 现在包含了单词 `aegilops`（一种侵入性杂草的属名） ➌。

#### *stable_sort*

`stable_sort` 算法对序列进行稳定排序。

算法就地对目标序列进行排序。相等元素保持其原始顺序。

```
void stable_sort([ep], rnd_begin, rnd_end, [comp]);
```

##### 参数

+   一个可选的 `std::execution` 执行策略，`ep`（默认值：`std::execution::seq`）

+   一对 `RandomAccessIterator`，`rnd_begin` 和 `rnd_end`，表示目标序列

+   一个可选的比较操作符，`comp`

##### 复杂度

**多对数线性** O(N log² N)，其中 N = `distance(rnd_begin, rnd_end)`。如果有额外内存可用，复杂度将减少到准线性。

##### 附加要求

目标序列的元素必须是可交换的、可移动构造的和可移动赋值的。

##### 示例

```
#include <algorithm>

enum class CharCategory { ➊
  Ascender,
  Normal,
  Descender
};

CharCategory categorize(char x) { ➋
  switch (x) {
 case 'g':
    case 'j':
    case 'p':
    case 'q':
    case 'y':
      return CharCategory::Descender;
    case 'b':
    case 'd':
    case 'f':
    case 'h':
    case 'k':
    case 'l':
    case 't':
      return CharCategory::Ascender;
  }
  return CharCategory::Normal;
}

bool ascension_compare(char x, char y) { ➌
  return categorize(x) < categorize(y);
}

TEST_CASE("stable_sort") {
  string word{ "outgrin" }; ➍
  stable_sort(word.begin(), word.end(), ascension_compare); ➎
  REQUIRE(word == "touring"); ➏
}
```

这个例子使用*升部字母*和*降部字母*对`string`进行排序。在排版学中，升部字母是指其一部分延伸到字体的平均线以上的字母。降部字母是指其一部分延伸到基线以下的字母。常见的降部字母有*g*、*j*、*p*、*q*和*y*。常见的升部字母有*b*、*d*、*f*、*h*、*k*、*l*和*t*。这个例子使用`stable_sort`，使得所有升部字母排在所有其他字母之前，所有降部字母排在所有其他字母之后。既不属于升部字母也不属于降部字母的字母则排在中间。作为一个`stable_sort`，具有相同升部/降部分类的字母的相对顺序不能发生变化。

你首先定义了一个`enum class`，名为`CharCategory`，它有三个可能的值：`Ascender`、`Normal`或`Descender` ➊。接下来，你定义了一个函数，用来将给定的字符分类到`CharCategory`中 ➋。（回想一下在第 50 页的“Switch 语句”部分，若不包含`break`，标签会“穿透”。）你还定义了一个`ascension_compare`函数，用于将两个给定的`char`对象转换为`CharCategory`对象，并通过`operator<`进行比较 ➌。由于`enum class`对象会隐式转换为`int`对象，并且你按预期的顺序定义了`CharCategory`，因此这将使得升部字母排在正常字母前面，正常字母排在降部字母前面。

在测试用例中，你初始化了一个包含单词`outgrin`的`string` ➍。接下来，你调用`stable_sort`，以该`string`作为目标序列，`ascension_compare`作为比较运算符 ➎。结果是，`word`现在包含了`touring` ➏。注意，`t`，唯一的升部字母，出现在所有正常字符之前（这些字符的顺序和`outgrin`中的顺序相同），而这些正常字符又出现在`g`之前，`g`是唯一的降部字母。

#### *partial_sort*

`partial_sort`算法将一个序列排序为两组。

如果是修改，算法会对目标序列中的前`(rnd_middle – rnd_first)`个元素进行排序，使得`rnd_begin`到`rnd_middle`中的所有元素都小于其余元素。如果是复制，算法会将前`min(distance(ipt_begin, ipt_end), distance(rnd_begin, rnd_end))`个已排序的元素放入目标序列，并返回一个指向目标序列末尾的迭代器。

基本上，部分排序允许你在不排序整个序列的情况下，找到排序序列中的前几个元素。例如，如果你有一个序列 D C B A，你可以对前两个元素进行部分排序，得到结果 A B D C。前两个元素和对整个序列进行排序的结果相同，但其余元素则没有进行排序。

```
void partial_sort([ep], rnd_begin, rnd_middle, rnd_end, [comp]);
RandomAccessIterator partial_sort_copy([ep], ipt_begin, ipt_end,
                                       rnd_begin, rnd_end, [comp]);
```

##### 参数

+   一个可选的`std::execution`执行策略，`ep`（默认值：`std::execution::seq`）

+   如果是修改，表示目标序列的三元组`rnd_begin`、`rnd_middle`和`rnd_end`的`RandomAccessIterator`

+   如果是复制，表示目标序列的`ipt_begin`和`ipt_end`一对，以及表示目标序列的`rnd_begin`和`rnd_end`一对

+   一个可选的比较运算符，`comp`

##### 复杂度

**准线性** O(N log N)，其中 N = `distance(rnd_begin, rnd_end) * log(distance(rnd_begin, rnd_middle)` 或 `distance(rnd_begin, rnd_end) * log(min(distance(rnd_begin, rnd_end), distance(ipt_begin, ipt_end))` 用于复制变体

##### 附加要求

目标序列的元素必须是可交换的、可移动构造的，并且可移动赋值的。

##### 示例

```
#include <algorithm>

bool ascension_compare(char x, char y) {
--snip--
}

TEST_CASE("partial_sort") {
  string word1{ "nectarous" }; ➊
  partial_sort(word1.begin(), word1.begin() + 4, word1.end()); ➋
  REQUIRE(word1 == "acentrous"); ➌

  string word2{ "pretanning" }; ➍
  partial_sort(word2.begin(), word2.begin() + 3, ➎
               word2.end(), ascension_compare);
  REQUIRE(word2 == "trepanning"); ➏
}
```

首先，你初始化一个包含单词`nectarous`的`string` ➊。接着，你用这个`string`作为目标序列，和第五个字母（`a`）作为`partial_sort`的第二个参数调用`partial_sort` ➋。结果是，序列现在包含单词`acentrous` ➌。注意，`acentrous`的前四个字母已经排序，并且它们小于序列中的剩余字符。

在第二个示例中，你初始化一个包含单词`pretanning`的`string` ➍，并将其用作`partial_sort`的目标序列 ➎。在这个示例中，你指定第四个字符（`t`）作为`partial_sort`的第二个参数，并使用`stable_sort`示例中的`ascension_compare`函数作为比较运算符。结果是，序列现在包含单词`trepanning` ➏。注意，前面三个字母是按`ascension_compare`排序的，并且`partial_sort`的第二个参数中的剩余字符都不小于前三个字符。

**注意**

*从技术上讲，前面的示例中的 REQUIRE 语句可能会在某些标准库实现中失败。因为`std::partial_sort`并不保证稳定性，结果可能会有所不同。*

#### *is_sorted*

`is_sorted`算法用于判断序列是否已排序。

如果目标序列按照`operator<`或（如果给定）`comp`排序，则该算法返回`true`。`is_sorted_until`算法返回指向第一个未排序元素的迭代器，或者如果目标序列已排序，则返回`rnd_end`。

```
bool is_sorted([ep], rnd_begin, rnd_end, [comp]);
ForwardIterator is_sorted_until([ep], rnd_begin, rnd_end, [comp]);
```

##### 参数

+   一个可选的`std::execution`执行策略，`ep`（默认值：`std::execution::seq`）

+   一对`RandomAccessIterator`，`rnd_begin`和`rnd_end`，表示目标序列

+   一个可选的比较运算符，`comp`

##### 复杂度

**线性** 该算法比较`distance(rnd_begin, rnd_end)`次。

##### 示例

```
#include <algorithm>

bool ascension_compare(char x, char y) {
--snip--
}

TEST_CASE("is_sorted") {
  string word1{ "billowy" }; ➊
  REQUIRE(is_sorted(word1.begin(), word1.end())); ➋

  string word2{ "floppy" }; ➌
  REQUIRE(word2.end() == is_sorted_until(word2.begin(), ➍
                                         word2.end(), ascension_compare));
}
```

首先，你构造一个包含单词`billowy`的`string` ➊。接着，你用这个`string`作为目标序列调用`is_sort`，它返回`true` ➋。

在第二个示例中，你构造一个包含单词`floppy`的`string` ➌。然后，你用这个`string`作为目标序列调用`is_sorted_until`，它返回`rnd_end`，因为该序列已排序 ➍。

#### *nth_element*

`nth_element`算法将序列中的特定元素放到其正确的排序位置。

这个部分排序算法以以下方式修改目标序列：`rnd_nth`指向的位置就像整个范围已排序一样。所有从`rnd_begin`到`rnd_nth-1`的位置的元素都小于`rnd_nth`。如果`rnd_nth == rnd_end`，则函数不执行任何操作。

```
bool nth_element([ep], rnd_begin, rnd_nth, rnd_end, [comp]);
```

##### 参数

+   一个可选的`std::execution`执行策略，`ep`（默认值：`std::execution::seq`）

+   一组三个`RandomAccessIterator`，`rnd_begin`、`rnd_nth`和`rnd_end`，表示目标序列

+   一个可选的比较运算符，`comp`

##### 复杂度

**线性** 该算法比较`distance(rnd_begin, rnd_end)`次。

##### 附加要求

目标序列的元素必须是可交换的、可移动构造的和可移动赋值的。

##### 示例

```
#include <algorithm>

TEST_CASE("nth_element") {
  vector<int> numbers{ 1, 9, 2, 8, 3, 7, 4, 6, 5 }; ➊
  nth_element(numbers.begin(), numbers.begin() + 5, numbers.end()); ➋
  auto less_than_6th_elem = [&elem=numbers[5]](int x) { ➌
    return x < elem;
  };
  REQUIRE(all_of(numbers.begin(), numbers.begin() + 5, less_than_6th_elem)); ➍
  REQUIRE(numbers[5] == 6 ); ➎
}
```

你首先构造一个包含数字序列 1 到 10 的`int`对象的`vector` ➊。然后，你使用这个`vector`作为目标序列，调用`nth_element` ➋。接着，你初始化一个名为`less_than_6th_elem`的 lambda，它使用`operator<`比较一个`int`与`numbers`中的第六个元素 ➌。这使得你可以检查所有第六个元素之前的元素是否都小于第六个元素 ➍。第六个元素是 6 ➎。

### 二分查找

*二分查找算法*假设目标序列已经排序。与在未指定序列上进行通用查找相比，这些算法具有理想的复杂度特性。本节中解释的每个算法都位于`<algorithm>`头文件中。

#### *lower_bound*

`lower_bound`算法在已排序的序列中找到一个分区。

该算法返回一个迭代器，指向元素`result`，它将序列划分，使得`result`之前的元素都小于`value`，而`result`及其后的所有元素不小于`value`。

```
ForwardIterator lower_bound(fwd_begin, fwd_end, value, [comp]);
```

##### 参数

+   一对`ForwardIterator`，`fwd_begin`和`fwd_end`，表示目标序列

+   一个用于划分目标序列的`value`

+   一个可选的比较运算符，`comp`

##### 复杂度

**对数** 如果提供了一个随机迭代器，复杂度为`O(log N)`，其中`N = distance(fwd_begin, fwd_end)`；否则，复杂度为`O(N)`

##### 附加要求

目标序列必须根据`operator<`或提供的`comp`进行排序。

##### 示例

```
#include <algorithm>

TEST_CASE("lower_bound") {
  vector<int> numbers{ 2, 4, 5, 6, 6, 9 }; ➊
  const auto result = lower_bound(numbers.begin(), numbers.end(), 5); ➋
  REQUIRE(result == numbers.begin() + 2); ➌
}
```

你首先构造一个`int`对象的`vector` ➊。然后，你使用这个`vector`作为目标序列，并提供`value`为`5`，调用`lower_bound` ➋。结果是第三个元素，`5` ➌。元素`2`和`4`小于`5`，而元素`5`、`6`、`6`和`9`不小于`5`。

#### *upper_bound*

`upper_bound`算法在已排序的序列中找到一个分区。

该算法返回一个迭代器，指向元素`result`，它是目标序列中大于`value`的第一个元素。

```
ForwardIterator upper_bound(fwd_begin, fwd_end, value, [comp]);
```

##### 参数

+   一对`ForwardIterator`，`fwd_begin`和`fwd_end`，表示目标序列

+   用于划分目标序列的`value`

+   一个可选的比较运算符，`comp`

##### 复杂度

**对数级** 如果提供一个随机迭代器，`O(log N)`，其中`N = distance (fwd_begin, fwd_end)`；否则，`O(N)`

##### 附加要求

目标序列必须按照`operator<`或提供的`comp`进行排序。

##### 示例

```
#include <algorithm>

TEST_CASE("upper_bound") {
 vector<int> numbers{ 2, 4, 5, 6, 6, 9 }; ➊
  const auto result = upper_bound(numbers.begin(), numbers.end(), 5); ➋
  REQUIRE(result == numbers.begin() + 3); ➌
}
```

首先构造一个`int`类型的`vector`对象 ➊。接着，调用`upper_bound`，将这个`vector`作为目标序列，`value`为`5` ➋。结果是第四个元素`6`，它是目标序列中大于`value`的第一个元素 ➌。

#### *equal_range*

`equal_range`算法在排序序列中查找一系列特定的元素。

算法返回一个`std::pair`的迭代器，表示等于`value`的半开区间。

```
ForwardIteratorPair equal_range(fwd_begin, fwd_end, value, [comp]);
```

##### 参数

+   一对`ForwardIterator`，`fwd_begin`和`fwd_end`，表示目标序列

+   要查找的`value`

+   一个可选的比较运算符，`comp`

##### 复杂度

**对数级** 如果提供一个随机迭代器，`O(log N)`，其中`N = distance (fwd_begin, fwd_end)`；否则，`O(N)`

##### 附加要求

目标序列必须按照`operator<`或提供的`comp`进行排序。

##### 示例

```
#include <algorithm>

TEST_CASE("equal_range") {
  vector<int> numbers{ 2, 4, 5, 6, 6, 9 }; ➊
  const auto[rbeg, rend] = equal_range(numbers.begin(), numbers.end(), 6); ➋
  REQUIRE(rbeg == numbers.begin() + 3); ➌
  REQUIRE(rend == numbers.begin() + 5); ➍
}
```

首先构造一个`int`类型的`vector`对象 ➊。接着，调用`equal_range`，将这个`vector`作为目标序列，`value`为`6` ➋。结果是一个表示匹配范围的迭代器对。第一个迭代器指向第四个元素 ➌，第二个迭代器指向第六个元素 ➍。

#### *binary_search*

`binary_search`算法在排序序列中查找特定元素。

如果范围包含`value`，算法返回`true`。具体来说，如果目标序列包含元素`x`，使得`x < value`和`value < x`都不成立，则返回`true`。如果提供了`comp`，则当目标序列包含元素`x`，且`comp(x, value)`和`comp(value, x)`都不成立时，返回`true`。

```
bool binary_search(fwd_begin, fwd_end, value, [comp]);
```

##### 参数

+   一对`ForwardIterator`，`fwd_begin`和`fwd_end`，表示目标序列

+   要查找的`value`

+   一个可选的比较运算符，`comp`

##### 复杂度

**对数级** 如果提供一个随机迭代器，`O(log N)`，其中`N = distance (fwd_begin, fwd_end)`；否则，`O(N)`

##### 附加要求

目标序列必须按照`operator<`或提供的`comp`进行排序。

##### 示例

```
#include <algorithm>

TEST_CASE("binary_search") {
  vector<int> numbers{ 2, 4, 5, 6, 6, 9 }; ➊
  REQUIRE(binary_search(numbers.begin(), numbers.end(), 6)); ➋
  REQUIRE_FALSE(binary_search(numbers.begin(), numbers.end(), 7)); ➌
}
```

首先构造一个`int`类型的`vector`对象 ➊。接着，调用`binary_search`，将这个`vector`作为目标序列，值为`6`。由于序列中包含 6，`binary_search`返回`true` ➋。当你调用`binary_search`并传入`7`时，它返回`false`，因为目标序列中不包含`7` ➌。

### 划分算法

一个*分区序列*包含两个连续的、不同的元素组。这些组不会混合，第二个不同组的第一个元素称为*分区点*。标准库包含用于分区序列、确定序列是否已分区以及查找分区点的算法。本节中解释的每个算法都在`<algorithm>`头文件中。

#### *is_partitioned*

`is_partitioned`算法用于确定一个序列是否已分区。

**注意**

*如果所有具有某些属性的元素都出现在没有这些属性的元素之前，则序列被认为是分区的。*

如果目标序列中所有对`pred`评估为`true`的元素都出现在其他元素之前，则算法返回`true`。

```
bool is_partitioned([ep], ipt_begin, ipt_end, pred);
```

##### 参数

+   一个可选的`std::execution`执行策略，`ep`（默认：`std::execution::seq`）

+   一对`InputIterator`对象，`ipt_begin`和`ipt_end`，表示目标序列

+   一个谓词，`pred`，用于确定组成员资格

##### 复杂度

**线性** 最多需要对`pred`进行`distance(ipt_begin, ipt_end)`次评估

##### 示例

```
#include <algorithm>

TEST_CASE("is_partitioned") {
  auto is_odd = [](auto x) { return x % 2 == 1; }; ➊

  vector<int> numbers1{ 9, 5, 9, 6, 4, 2 }; ➋
  REQUIRE(is_partitioned(numbers1.begin(), numbers1.end(), is_odd)); ➌

  vector<int> numbers2{ 9, 4, 9, 6, 4, 2 }; ➍
  REQUIRE_FALSE(is_partitioned(numbers2.begin(), numbers2.end(), is_odd)); ➎
}
```

你首先构造一个名为`is_odd`的 lambda，如果给定的数字是奇数，则返回`true` ➊。接着，你构造一个`int`对象的`vector` ➋，并使用这个`vector`作为目标序列，`is_odd`作为谓词调用`is_partitioned`。因为序列中的所有奇数都排在偶数前面，所以`is_partitioned`返回`true` ➌。

然后，你构造另一个`int`对象的`vector` ➍，并再次使用这个`vector`作为目标序列，`is_odd`作为谓词调用`is_partitioned`。因为该序列并没有把所有的奇数放在偶数前面（4 是偶数，且排在第二个 9 之前），所以`is_partitioned`返回`false` ➎。

#### *partition*

`partition`算法用于对序列进行分区。

该算法会修改目标序列，使其根据`pred`进行分区。它返回分区点。元素的原始顺序不一定会被保留。

```
ForwardIterator partition([ep], fwd_begin, fwd_end, pred);
```

##### 参数

+   一个可选的`std::execution`执行策略，`ep`（默认：`std::execution::seq`）

+   一对`ForwardIterator`，`fwd_begin`和`fwd_end`，表示目标序列

+   一个谓词，`pred`，用于确定组成员资格

##### 复杂度

**线性** 最多需要对`pred`进行`distance(fwd_begin, fwd_end)`次评估

##### 附加要求

目标序列的元素必须是可交换的。

##### 示例

```
#include <algorithm>

TEST_CASE("partition") {
  auto is_odd = [](auto x) { return x % 2 == 1; }; ➊
  vector<int> numbers{ 1, 2, 3, 4, 5 }; ➋
  const auto partition_point = partition(numbers.begin(),
                                         numbers.end(), is_odd); ➌
  REQUIRE(is_partitioned(numbers.begin(), numbers.end(), is_odd)); ➍
  REQUIRE(partition_point == numbers.begin() + 3); ➎
}
```

你首先构造一个名为`is_odd`的 lambda，如果给定的数字是`odd`（奇数）则返回`true` ➊。接着，你构造一个`int`对象的`vector` ➋，并使用这个`vector`作为目标序列，`is_odd`作为谓词调用`partition`。你将结果分区点赋值给`partition_point` ➌。

当你在目标序列上调用`is_partitioned`，并以`is_odd`作为谓词时，它返回`true` ➍。根据算法的规范，*你不能依赖于组内的顺序*，但是`partition_point`将始终是第四个元素，因为目标序列包含三个奇数 ➎。

#### *partition_copy*

`partition_copy`算法对一个序列进行分区。

该算法通过在每个元素上评估`pred`来对目标序列进行分区。所有`true`元素复制到`opt_true`中，所有`false`元素复制到`opt_false`中。

```
ForwardIteratorPair partition_copy([ep], ipt_begin, ipt_end,
                                         opt_true, opt_false, pred);
```

##### 参数

+   一个可选的`std::execution`执行策略，`ep`（默认值：`std::execution::seq`）

+   一对`InputIterator`对象，`ipt_begin`和`ipt_end`，表示目标序列

+   一个`OutputIterator`，`opt_true`，用于接收`true`元素的副本

+   一个`OutputIterator`，`opt_false`，用于接收`false`元素的副本

+   一个谓词，`pred`，用于确定组成员资格

##### 复杂度

**线性** 精确地进行`distance(ipt_begin, ipt_end)`次`pred`评估

##### 附加要求

+   目标序列的元素必须是可复制赋值的。

+   输入和输出范围不能重叠。

##### 示例

```
#include <algorithm>

TEST_CASE("partition_copy") {
  auto is_odd = [](auto x) { return x % 2 == 1; }; ➊
  vector<int> numbers{ 1, 2, 3, 4, 5 }, odds, evens; ➋
  partition_copy(numbers.begin(), numbers.end(),
                 back_inserter(odds), back_inserter(evens), is_odd); ➌
  REQUIRE(all_of(odds.begin(), odds.end(), is_odd)); ➍
  REQUIRE(none_of(evens.begin(), evens.end(), is_odd)); ➎
}
```

首先构造一个名为`is_odd`的 lambda，如果给定的数字是`odd`（奇数），则返回`true` ➊。接下来，构造一个包含从 1 到 5 的`int`对象的`vector`，以及两个空的`vector`对象，分别名为`odds`和`evens` ➋。然后，使用`partition_copy`，将`numbers`作为目标序列，一个`back_inserter`插入到`odds`作为`true`元素的输出，一个`back_inserter`插入到`evens`作为`false`元素的输出，`is_odd`作为谓词 ➌。结果是，所有`odds`中的元素都是奇数 ➍，而`evens`中的元素没有奇数 ➎。

#### *stable_partition*

`stable_partition`算法稳定地对序列进行分区。

**注意**

*稳定分区可能比不稳定分区需要更多的计算，因此用户可以选择。*

该算法会改变目标序列，使其根据`pred`进行分区，并返回分区点。元素的原始顺序将被保留。

```
BidirectionalIterator stable_partition([ep], bid_begin, bid_end, pred);
```

##### 参数

+   一个可选的`std::execution`执行策略，`ep`（默认值：`std::execution::seq`）

+   一对`BidirectionalIterator`，`bid_begin`和`bid_end`，表示目标序列

+   一个谓词，`pred`，用于确定组成员资格

##### 复杂度

**准线性** `O(N log N)`次交换，其中`N = distance(bid_begin, bid_end)`，或者如果有足够的内存，`O(N)`次交换。

##### 附加要求

目标序列的元素必须是可交换的、可移动构造的，并且可以进行移动赋值。

##### 示例

```
#include <algorithm>

TEST_CASE("stable_partition") {
  auto is_odd = [](auto x) { return x % 2 == 1; }; ➊
  vector<int> numbers{ 1, 2, 3, 4, 5 }; ➋
  stable_partition(numbers.begin(), numbers.end(), is_odd); ➌
  REQUIRE(numbers == vector<int>{ 1, 3, 5, 2, 4 }); ➍
}
```

首先，你构造一个名为`is_odd`的 lambda，它返回`true`，如果给定的数字是`odd` ➊。接下来，你构造一个`int`类型的`vector`对象 ➋，并使用`stable_partition`，以这个`vector`作为目标序列，`is_odd`作为谓词 ➌。结果是`vector`包含元素 1、3、5、2、4，因为这是唯一能够在保持原始组内顺序的情况下划分这些数字的方法 ➍。

### 合并算法

*合并算法*将两个已排序的目标序列合并，使得结果序列包含两个目标序列的副本，并且也是排序的。本节中解释的每个算法都位于`<algorithm>`头文件中。

#### *合并*

`merge`算法合并两个已排序的序列。

该算法将两个目标序列复制到目标序列中。如果提供了`operator<`或`comp`，目标序列将根据这些进行排序。

```
OutputIterator merge([ep], ipt_begin1, ipt_end1,
                     ipt_begin2, ipt_end2, opt_result, [comp]);
```

##### 参数

+   一个可选的`std::execution`执行策略，`ep`（默认值：`std::execution::seq`）

+   两对`InputIterator`，`ipt_begin`和`ipt_end`，表示目标序列

+   一个`OutputIterator`，`opt_result`，表示目标序列

+   一个谓词，`pred`，用于确定组成员资格

##### 复杂度

**线性** 最多进行`N-1`次比较，其中`N = distance(ipt_begin1, ipt_end1) + distance(ipt_begin2, ipt_end2)`

##### 附加要求

如果提供了`operator<`或`comp`，则目标序列必须根据这些进行排序。

##### 示例

```
#include <algorithm>

TEST_CASE("merge") {
  vector<int> numbers1{ 1, 4, 5 }, numbers2{ 2, 3, 3, 6 }, result; ➊
  merge(numbers1.begin(), numbers1.end(),
        numbers2.begin(), numbers2.end(),
        back_inserter(result)); ➋
  REQUIRE(result == vector<int>{ 1, 2, 3, 3, 4, 5, 6 }); ➌
}
```

你构造三个`vector`对象：两个包含已排序的`int`对象，另一个为空➊。接下来，你将非空的`vector`与空的`vector`合并，并使用空的`vector`作为目标序列，利用`back_inserter` ➋。`result`包含了原始序列中所有元素的副本，并且它本身也已排序 ➌。

### 极值算法

一些被称为*极值算法*的算法，用于确定最小值和最大值元素，或者限制元素的最小值或最大值。本节中解释的每个算法都位于`<algorithm>`头文件中。

#### *最小值和最大值*

`min`或`max`算法用于确定序列的极值。

这些算法使用`operator<`或`comp`，并返回最小值（`min`）或最大值（`max`）对象。`minmax`算法同时返回这两个值，作为一个`std::pair`，其中`first`为最小值，`second`为最大值。

```
T min(obj1, obj2, [comp]);
T min(init_list, [comp]);
T max(obj1, obj2, [comp]);
T max(init_list, [comp]);
Pair minmax(obj1, obj2, [comp]);
Pair minmax(init_list, [comp]);
```

##### 参数

+   两个对象，`obj1`和`obj2`，或者

+   一个初始化列表，`init_list`，表示要比较的对象

+   一个可选的比较函数，`comp`

##### 复杂度

**常数或线性** 对于需要`obj1`和`obj2`的重载，恰好有一个比较。对于初始化列表，最多进行`N-1`次比较，其中`N`是初始化列表的长度。对于`minmax`，给定初始化列表，比较次数将增长到`3/2 N`。

##### 附加要求

元素必须是可复制构造的，并且可以使用给定的比较方法进行比较。

##### 示例

```
#include <algorithm>

TEST_CASE("max and min") {
 auto length_compare = [](const auto& x1, const auto& x2) { ➊
    return x1.length() < x2.length();
  };

string undisc="undiscriminativeness", vermin="vermin";
  REQUIRE(min(undisc, vermin, length_compare) == "vermin"); ➋

string maxim="maxim", ultra="ultramaximal";
  REQUIRE(max(maxim, ultra, length_compare) == "ultramaximal"); ➌

string mini="minimaxes", maxi="maximin";
  const auto result = minmax(mini, maxi, length_compare); ➍
  REQUIRE(result.first == maxi); ➎
  REQUIRE(result.second == mini); ➏
}
```

你首先初始化一个名为`length_compare`的 lambda，它使用`operator<`来比较两个输入的长度 ➊。接着，你使用`min`来确定*undiscriminativeness*和*vermin*哪个长度较小 ➋，并使用`max`来确定*maxim*和*ultramaximal*哪个长度较大 ➌。最后，你使用`minmax`来确定*minimaxes*和*maximin*哪个具有最小和最大长度 ➍。结果是一个对 ➎➏。

#### *min_element 和 max_element*

`min_element`或`max_element`算法确定一个序列的极值。

这些算法使用`operator<`或`comp`，并返回指向最小值（`min_element`）或最大值（`max_element`）的迭代器。`minimax_element`算法同时返回最小值和最大值，作为一个`std::pair`，`first`表示最小值，`second`表示最大值。

```
ForwardIterator min_element([ep], fwd_begin, fwd_end, [comp]);
ForwardIterator max_element([ep], fwd_begin, fwd_end, [comp]);
Pair minmax_element([ep], fwd_begin, fwd_end, [comp]);
```

##### 参数

+   一个可选的`std::execution`执行策略，`ep`（默认值：`std::execution::seq`）

+   一对`ForwardIterator`，`fwd_begin`和`fwd_end`，表示目标序列

+   一个可选的比较函数，`comp`

##### 复杂度

**线性** 对于`max`和`min`，最多进行`N-1`次比较，其中`N=distance(fwd_begin, fwd_end)`；对于`minmax`，则为`3/2 N`

##### 附加要求

元素必须能够使用给定的操作进行比较。

##### 示例

```
#include <algorithm>

TEST_CASE("min and max element") {
  auto length_compare = [](const auto& x1, const auto& x2) { ➊
    return x1.length() < x2.length();
  };

  vector<string> words{ "civic", "deed", "kayak",  "malayalam" }; ➋

  REQUIRE(*min_element(words.begin(), words.end(),
                       length_compare) == "deed"); ➌
  REQUIRE(*max_element(words.begin(), words.end(),
                       length_compare) == "malayalam"); ➍

  const auto result = minmax_element(words.begin(), words.end(),
                                     length_compare); ➎
  REQUIRE(*result.first == "deed"); ➏
  REQUIRE(*result.second == "malayalam"); ➐
}
```

你首先初始化一个名为`length_compare`的 lambda，它使用`operator<`来比较两个输入的长度 ➊。接着，你初始化一个包含四个单词的`string`对象`vector`，名为`words` ➋。你使用`min_element`来确定这些单词中最小的那个，通过将它作为目标序列，并将`length_compare`作为比较函数（`deed`） ➌，然后使用`max_element`来确定最大的单词（`malayalam`） ➍。最后，你使用`minmax_element`，它返回最小值和最大值，作为一个`std::pair` ➎。`first`元素表示最短的`word` ➏，`second`元素表示最长的`word` ➐。

#### *clamp*

`clamp`算法对值进行约束。

该算法使用`operator<`或`comp`来判断`obj`是否在`low`到`high`的范围内。如果在范围内，算法直接返回`obj`；否则，如果`obj`小于`low`，则返回`low`；如果`obj`大于`high`，则返回`high`。

```
T& clamp(obj, low, high, [comp]);
```

##### 参数

+   一个对象，`obj`

+   一个`low`和`high`对象

+   一个可选的比较函数，`comp`

##### 复杂度

**常数** 最多进行两次比较

##### 附加要求

这些对象必须能够使用给定的操作进行比较。

##### 示例

```
#include <algorithm>

TEST_CASE("clamp") {
  REQUIRE(clamp(9000, 0, 100) == 100); ➊
  REQUIRE(clamp(-123, 0, 100) == 0); ➋
  REQUIRE(clamp(3.14, 0., 100.) == Approx(3.14)); ➌
}
```

在第一个示例中，你将`9000`限制在从 0 到 100 的区间内。因为 9000 > 100，所以结果是`100` ➊。在第二个示例中，你将`-123`限制在同一区间内。因为−123 < 0，所以结果是`0` ➋。最后，你将`3.14`限制在区间内，由于它在区间内，因此结果是`3.14` ➌。

### 数值操作

`<numeric>` 头文件在 第十二章 中讨论过，你在那时学习了它的数学类型和函数。它还提供了非常适合数值操作的算法。本节介绍了其中的许多算法。本节中解释的每个算法都在 `<numeric>` 头文件中。

#### *常用操作符*

一些标准库的数值操作允许你传递操作符以自定义行为。为方便起见，`<functional>` 头文件提供了以下类模板，通过 `operator(T x, T y)` 暴露各种二元算术操作：

+   `plus<T>` 实现加法 `x + y`。

+   `minus<T>` 实现减法 `x - y`。

+   `multiplies<T>` 实现乘法 `x * y`。

+   `divides<T>` 实现除法 `x / y`。

+   `modulus<T>` 实现模运算 `x % y`。

例如，你可以使用 `plus` 模板来加两个数字，像这样：

```
#include <functional>

TEST_CASE("plus") {
  plus<short> adder; ➊
  REQUIRE(3 == adder(1, 2)); ➋
  REQUIRE(3 == plus<short>{}(1,2)); ➌
}
```

首先，你实例化一个名为 `adder` 的 `plus` ➊，然后用 `1` 和 `2` 调用它，结果是 `3` ➋。你也可以完全省略变量，直接使用新构造的 `plus` 来实现相同的结果 ➌。

**注意**

*通常，除非你正在使用需要这些操作符类型的泛型代码，否则不会使用它们。*

#### *iota*

`iota` 算法将序列填充为递增的值。

算法从 `start` 开始，依次将递增值赋给目标序列。

```
void iota(fwd_begin, fwd_end, start);
```

##### 参数

+   一对迭代器 `fwd_begin` 和 `fwd_end`，表示目标序列

+   一个 `start` 值

##### 复杂度

**线性** `N` 次增量和赋值，其中 `N=distance(fwd_begin, fwd_end)`

##### 附加要求

对象必须能够赋值给 `start`。

##### 示例

```
#include <numeric>
#include <array>

TEST_CASE("iota") {
  array<int, 3> easy_as; ➊
  iota(easy_as.begin(), easy_as.end(), 1); ➋
  REQUIRE(easy_as == array<int, 3>{ 1, 2, 3 }); ➌
}
```

首先，你初始化一个长度为 `3` 的 `int` 对象数组 ➊。接着，你调用 `iota`，将 `array` 作为目标序列，`1` 作为 `start` 值 ➋。结果是 `array` 包含元素 1、2 和 3 ➌。

#### *累加*

`accumulate` 算法按顺序折叠一个序列。

**注意**

*折叠一个序列意味着对序列的元素应用特定操作，同时将累积结果传递给下一个操作。*

该算法将 `op` 应用于 `start` 和目标序列的第一个元素。然后它将结果与目标序列的下一个元素再次应用 `op`，以此类推，直到遍历目标序列中的每个元素。大致来说，这个算法将目标序列的元素和 `start` 值相加，并返回结果。

```
T accumulate(ipt_begin, ipt_end, start, [op]);
```

##### 参数

+   一对迭代器 `ipt_begin` 和 `ipt_end`，表示目标序列

+   一个 `start` 值

+   一个可选的二元操作符 `op`，默认为 `plus`

##### 复杂度

**线性** `N` 次应用 `op`，其中 `N=distance(ipt_begin, ipt_end)`

##### 附加要求

目标序列的元素必须是可复制的。

##### 示例

```
#include <numeric>

TEST_CASE("accumulate") {
  vector<int> nums{ 1, 2, 3 }; ➊
  const auto result1 = accumulate(nums.begin(), nums.end(), -1); ➋
  REQUIRE(result1 == 5); ➌

  const auto result2 = accumulate(nums.begin(), nums.end(),
                                  2, multiplies<>()); ➍
  REQUIRE(result2 == 12); ➎
}
```

你首先初始化一个长度为`3`的`vector`类型的`int`对象 ➊。接着，你使用`vector`作为目标序列，并将`-1`作为`start`值调用`accumulate` ➋。结果是 −1 + 1 + 2 + 3 = 5 ➌。

在第二个示例中，你使用相同的目标序列，但`start`值为`2`，操作符改为`multiplies`。结果是 2 * 1 * 2 * 3 = 12 ➎。

#### *reduce*

`reduce`算法对一个序列进行折叠（不一定按顺序）。

该算法与`accumulate`相同，只是它接受一个可选的`execution`并且不保证操作符应用的顺序。

```
T reduce([ep], ipt_begin, ipt_end, start, [op]);
```

##### 参数

+   一个可选的`std::execution`执行策略，`ep`（默认为`std::execution::seq`）

+   一对迭代器，`ipt_begin`和`ipt_end`，表示目标序列

+   一个`start`值

+   一个可选的二元操作符，`op`，默认为`plus`

##### 复杂度

**线性** `N` 次`op`应用，其中`N=distance(ipt_begin, ipt_end)`

##### 附加要求

+   如果省略了`ep`，元素必须是可移动的。

+   如果提供了`ep`，元素必须是可复制的。

##### 示例

```
#include <numeric>

TEST_CASE("reduce") {
  vector<int> nums{ 1, 2, 3 }; ➊
  const auto result1 = reduce(nums.begin(), nums.end(), -1); ➋
  REQUIRE(result1 == 5); ➌

  const auto result2 = reduce(nums.begin(), nums.end(),
                                  2, multiplies<>()); ➍
  REQUIRE(result2 == 12); ➎
}
```

你首先初始化一个长度为`3`的`vector`类型的`int`对象 ➊。接着，你使用`vector`作为目标序列，并将`-1`作为`start`值调用`reduce` ➋。结果是 −1 + 1 + 2 + 3 = 5 ➌。

在第二个示例中，你使用相同的目标序列，但`start`值为`2`，操作符改为`multiplies`。结果是 2 * 1 * 2 * 3 = 12 ➎。

#### *inner_product*

`inner_product`算法计算两个序列的内积。

**注意**

*内积（或点积）是与一对序列相关的标量值。*

该算法将`op2`应用于目标序列中每一对对应元素，并使用`op1`将它们与`start`相加。

```
T inner_product([ep], ipt_begin1, ipt_end1, ipt_begin2, start, [op1], [op2]);
```

##### 参数

+   一对迭代器，`ipt_begin1`和`ipt_end1`，表示目标序列 1

+   一个迭代器，`ipt_begin2`，表示目标序列 2

+   一个`start`值

+   两个可选的二元操作符，`op1`和`op2`，默认为`plus`和`multiply`

##### 复杂度

**线性** `N` 次`op1`和`op2`应用，其中`N=distance(ipt_begin1, ipt_end1)`

##### 附加要求

元素必须是可复制的。

##### 示例

```
#include <numeric>

TEST_CASE("inner_product") {
  vector<int> nums1{ 1, 2, 3, 4, 5 }; ➊
  vector<int> nums2{ 1, 0,-1, 0, 1 }; ➋
  const auto result = inner_product(nums1.begin(), nums1.end(),
                                    nums2.begin(), 10); ➌
  REQUIRE(result == 13); ➍
}
```

你首先初始化两个`vector`类型的`int`对象 ➊ ➋。接着，你使用这两个`vector`对象作为目标序列，并将`10`作为`start`值调用`inner_product` ➌。结果是 10 + 1 * 1 + 2 * 0 + 3 * 1 + 4 * 0 + 4 * 1 = 13 ➍。

#### *adjacent_difference*

`adjacent_difference`算法生成相邻元素的差值。

**注意**

*相邻差值是对每一对邻近元素应用某个操作的结果。*

该算法将目标序列的第一个元素设置为目的序列的第一个元素。对于每个后续元素，它将`op`应用于前一个元素和当前元素，并将返回值写入`result`。该算法返回目的序列的结尾。

```
OutputIterator adjacent_difference([ep], ipt_begin, ipt_end, result, [op]);
```

##### 参数

+   一对迭代器，`ipt_begin` 和 `ipt_end`，表示目标序列。

+   一个迭代器，`result`，表示目标序列。

+   一个可选的二元操作符，`op`，默认为 `minus`。

##### 复杂度

**线性** `N-1` 次 `op` 应用，其中 `N=distance(ipt_begin, ipt_end)`

##### 附加要求

+   如果省略 `ep`，元素必须是可移动的。

+   如果你提供了 `ep`，元素必须是可复制的。

##### 示例

```
#include <numeric>

TEST_CASE("adjacent_difference") {
  vector<int> fib{ 1, 1, 2, 3, 5, 8 }, fib_diff; ➊
  adjacent_difference(fib.begin(), fib.end(), back_inserter(fib_diff)); ➋
  REQUIRE(fib_diff == vector<int>{ 1, 0, 1, 1, 2, 3 }); ➌
}
```

你首先初始化一个 `int` 类型的 `vector` 对象，一个包含斐波那契数列的前六个数字，另一个为空 ➊。接下来，你调用 `adjacent_difference`，将两个 `vector` 对象作为目标序列 ➋。结果如预期所示：第一个元素等于斐波那契数列的第一个元素，后续元素是相邻差（1 – 1 = 0），（2 – 1 = 1），（3 – 2 = 1），（5 – 3 = 2），（8 – 5 = 3） ➌。

#### *partial_sum*

`partial_sum` 算法生成部分和。

该算法将累加器设置为目标序列的第一个元素。对于目标序列中的每个后续元素，算法将该元素添加到累加器中，然后将累加器写入目标序列。该算法返回目标序列的末尾。

```
OutputIterator partial_sum(ipt_begin, ipt_end, result, [op]);
```

##### 参数

+   一对迭代器，`ipt_begin` 和 `ipt_end`，表示目标序列。

+   一个迭代器，`result`，表示目标序列。

+   一个可选的二元操作符，`op`，默认为 `plus`。

##### 复杂度

**线性** `N-1` 次 `op` 应用，其中 `N=distance(ipt_begin, ipt_end)`

##### 示例

```
#include <numeric>

TEST_CASE("partial_sum") {
  vector<int> num{ 1, 2, 3, 4 }, result; ➊
  partial_sum(num.begin(), num.end(), back_inserter(result)); ➋
  REQUIRE(result == vector<int>{ 1, 3, 6, 10 }); ➌
}
```

你首先初始化两个 `int` 类型的 `vector` 对象，一个名为 `num` 包含前四个计数值，另一个名为 `result` 是空的 ➊。接下来，你调用 `partial_sum`，以 `num` 作为目标序列，`result` 作为目的地 ➋。第一个元素等于目标序列的第一个元素，后续元素是部分和（1 + 2 = 3），（3 + 3 = 6），（6 + 4 = 10） ➌。

#### *其他算法*

为了防止一章内容过长，许多算法被省略。本节对它们进行了概述。

##### （最大）堆操作

长度为 *N* 的范围是最大堆，如果对于所有 0 < *i* < *N*，! Image 处的元素（向下取整）不会小于 *i* 处的元素。这些结构在需要快速查找最大元素和插入元素的情况下具有较强的性能特点。

`<algorithm>` 头文件包含了许多有助于处理此类范围的函数，例如 表 18-1 中的那些。详情请参见 [alg.heap.operations]。

**表 18-1：** `<algorithm>` 头文件中的堆相关算法

| **算法** | **描述** |
| --- | --- |
| `is_heap` | 检查一个范围是否是最大堆 |
| `is_heap_until` | 查找最大堆的最大子范围 |
| `make_heap` | 创建一个最大堆 |
| `push_heap` | 添加一个元素 |
| `pop_heap` | 移除最大元素 |
| `sort_heap` | 将最大堆转换为已排序范围 |

##### 对已排序范围的集合操作

`<algorithm>` 头文件包含对已排序范围进行集合操作的函数，如表 18-2 所示。详情请参见[alg.set.operations]。

**表 18-2：** `<algorithm>` 头文件中的集合相关算法

| **算法** | **描述** |
| --- | --- |
| `includes` | 如果一个范围是另一个范围的子集，则返回`true` |
| `set_difference` | 计算两个集合的差集 |
| `set_intersection` | 计算两个集合的交集 |
| `set_symmetric_difference` | 计算两个集合的对称差集 |
| `set_union` | 计算两个集合的并集 |

##### 其他数值算法

`<numeric>` 头文件包含了除“数值运算”部分介绍的函数之外的多个其他函数。表 18-3 列出了它们。详情请参见[numeric.ops]。

**表 18-3：** `<numeric>` 头文件中的附加数值算法

| **算法** | **描述** |
| --- | --- |
| `exclusive_scan` | 类似于`partial_sum`，但将第`i`个元素排除在第`i`个和之外 |
| `inclusive_scan` | 类似于`partial_sum`，但不按顺序执行，并且需要关联操作 |
| `transform_reduce` | 应用一个函数对象；然后进行不按顺序的归约 |
| `transform_exclusive_scan` | 应用一个函数对象；然后计算排他性扫描 |
| `transform_inclusive_scan` | 应用一个函数对象；然后计算包含性扫描 |

##### 内存操作

`<memory>` 头文件包含了多个低级别的函数，用于处理未初始化的内存。表 18-4 列出了它们。详情请参见[memory.syn]。

**表 18-4：** `<memory>` 头文件中用于未初始化内存的操作

| **算法** | **描述** |
| --- | --- |
| `uninitialized_copy``uninitialized_copy_n``uninitialized_fill``uninitialized_fill_n` | 将对象复制到未初始化的内存中 |
| `uninitialized_move``uninitialized_move_n` | 将对象移动到未初始化的内存中 |
| `uninitialized_default_construct``uninitialized_default_construct_n``uninitialized_value_construct``uninitialized_value_construct_n` | 在未初始化的内存中构造对象 |
| `destroy_at``destroy``destroy_n` | 销毁对象 |

### Boost Algorithm

Boost Algorithm 是一个庞大的算法库，部分与标准库重叠。由于篇幅限制，表 18-5 仅列出了标准库中未包含的算法的快速参考。有关更多信息，请参阅 Boost Algorithm 文档。

**表 18-5：** Boost Algorithm 中的附加算法

| **算法** | **描述** |
| --- | --- |
| `boyer_moore``boyer_moore_horspool``knuth_morris_pratt` | 用于搜索值序列的快速算法 |
| `hex``unhex` | 写入/读取十六进制字符 |
| `gather` | 接受一个序列并将满足谓词的元素移动到给定位置 |
| `find_not` | 查找序列中第一个不等于某个值的元素 |
| `find_backward` | 类似于 `find`，但从后向前查找 |
| `is_partitioned_until` | 返回从目标序列的第一个元素开始的最大分区子序列的结束迭代器 |
| `apply_permutation``apply_reverse_permutation` | 接受一个项目序列和一个顺序序列，并根据顺序序列重新排列项目序列 |
| `is_palindrome` | 如果序列是回文，则返回 `true` |

**关于范围的说明**

第八章介绍了作为基于范围的 `for` 循环的一部分的范围表达式。回顾这一讨论，范围是一个概念，它公开 `begin` 和 `end` 方法来返回迭代器。由于你可以对迭代器施加要求以支持某些操作，因此你可以对范围施加传递性要求，使其提供某些迭代器。每个算法都有特定的操作要求，这些要求反映在它们所需的迭代器类型中。由于你可以用范围来封装算法输入序列的要求，因此你必须理解各种范围类型，以理解每个算法的约束。

和概念一样，范围尚未正式成为 C++ 的一部分。尽管理解范围、迭代器和算法之间的关系仍然会带来巨大的好处，但也有两个缺点。首先，算法仍然需要迭代器作为输入参数，因此即使有了范围，你仍然需要手动提取迭代器（例如，使用 `begin` 和 `end`）。其次，像其他函数模板一样，当你违反算法的操作要求时，可能会得到极其糟糕的错误信息。

正在进行将范围正式引入语言的工作。事实上，概念和范围很可能会同时进入 C++ 标准，因为它们的结合非常自然。

如果你想尝试实现一个可能的范围操作，请参考 Boost Range。

**进一步阅读**

+   *ISO 国际标准 ISO/IEC (2017) — 编程语言 C++*（国际标准化组织；瑞士日内瓦；*[`isocpp.org/std/the-standard/`](https://isocpp.org/std/the-standard/)*）

+   *《C++标准库：教程与参考》*，第 2 版，尼科莱·约苏蒂斯著（Addison-Wesley Professional, 2012）

+   维克托·亚当奇克的《算法复杂性》([*https://www.cs.cmu.edu/~adamchik/15-121/lectures/Algorithmic%20Complexity/complexity.html*](https://www.cs.cmu.edu/~adamchik/15-121/lectures/Algorithmic%20Complexity/complexity.html))

+   *《Boost C++库》*，第 2 版，博里斯·谢林著（XML Press, 2014）
