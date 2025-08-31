## 第十八章：STRINGS**

*如果你用一个人能理解的语言和他交谈，那会打动他的头脑。如果你用他的语言和他说话，那会打动他的心。*

—纳尔逊·曼德拉*

![Image](img/common.jpg)

STL 提供了一种专门的 *字符串容器* 用于处理人类语言数据，如单词、句子和标记语言。`std::basic_string` 是一个类模板，可以根据字符串的底层字符类型进行特化，位于 `<string>` 头文件中。作为一个顺序容器，`basic_string` 本质上类似于 `vector`，但具有一些特殊的功能，用于处理语言数据。

STL 的 `basic_string` 在安全性和功能性上相较于 C 风格的字符串或空终止字符串有了显著提升，而且由于人类语言数据充斥着现代程序，你很可能会发现 `basic_string` 是不可或缺的。

### std::string

STL 提供了四种 `basic_string` 特化形式，在 `<string>` 头文件中定义。每种特化形式使用你在第二章中学习到的基本字符类型之一来实现字符串：

+   `std::string` 用于 `char`，适用于像 ASCII 这样的字符集。

+   `std::wstring` 用于 `wchar_t`，其大小足以包含实现区域设置中的最大字符。

+   `std::u16string` 用于 `char16_t`，适用于像 UTF-16 这样的字符集。

+   `std::u32string` 用于 `char32_t`，适用于像 UTF-32 这样的字符集。

你将使用具有适当底层类型的特化形式。因为这些特化形式具有相同的接口，本章中的所有示例将使用 `std::string`。

#### *构造*

`basic_string` 容器接受三个模板参数：

+   底层字符类型，`T`

+   底层类型的特性，`Traits`

+   分配器，`Alloc`

在这些参数中，只有 `T` 是必需的。STL 中的 `std::char_traits` 模板类位于 `<string>` 头文件中，抽象了字符和字符串操作，隐藏了底层字符类型的细节。此外，除非你计划支持自定义字符类型，否则你不需要实现自己的类型特性，因为 `char_traits` 已经为 `char`、`wchar_t`、`char16_t` 和 `char32_t` 提供了特化。如果标准库为某个类型提供了特化，除非你需要某种特殊行为，否则不必自己实现。

合起来，一个 `basic_string` 特化形式看起来像这样，其中 `T` 是字符类型：

```
std::basic_string<T, Traits=std::char_traits<T>, Alloc=std::allocator<T>>
```

**注意**

*在大多数情况下，你将处理其中一个预定义的特化形式，尤其是 `string` 或 `wstring`。然而，如果你需要自定义分配器，你将需要适当地特化 `basic_string`。*

`basic_string<T>` 容器支持与 `vector<T>` 相同的构造函数，并提供额外的便利构造函数用于转换 C 风格字符串。换句话说，`string` 支持 `vector<char>` 的构造函数，`wstring` 支持 `vector<wchar_t>` 的构造函数，依此类推。与 `vector` 一样，除了当你确实想要使用初始化列表时，所有 `basic_string` 的构造函数都需要使用圆括号。

你可以默认构造一个空字符串，或者如果你想用重复的字符填充一个`string`，你可以使用填充构造函数，通过传递一个`size_t`和一个`char`，正如清单 15-1 所示。

```
#include <string>
TEST_CASE("std::string supports constructing") {
  SECTION("empty strings") {
    std::string cheese; ➊
    REQUIRE(cheese.empty()); ➋
  }
  SECTION("repeated characters") {
    std::string roadside_assistance(3, 'A'); ➌
    REQUIRE(roadside_assistance == "AAA"); ➍
  }
}
```

*清单 15-1：`string`的默认构造函数和填充构造函数*

在你默认构造一个`string` ➊之后，它不包含任何元素 ➋。如果你想用重复的字符填充`string`，你可以使用填充构造函数，通过传入你想要填充的元素个数及其值 ➌。这个例子将一个字符串填充了三个`A`字符 ➍。

**注意**

*你将在本章稍后了解 std::string 的比较操作符==。因为你通常通过原始指针或原始数组来处理 C 风格字符串，所以操作符==只有在给定相同对象时才返回 true。然而，对于 std::string，操作符==如果内容相同则返回 true。如清单 15-1 所示，即使其中一个操作数是 C 风格字符串字面量，比较也能正常工作。*

`string`构造函数还提供了两个基于`const char*`的构造函数。如果传入的参数指向一个以 null 结尾的字符串，`string`构造函数可以自行确定输入的长度。如果指针*不*指向一个以 null 结尾的字符串，或者你只想使用`string`的前一部分，你可以传递一个长度参数，告诉`string`构造函数需要复制多少元素，正如清单 15-2 所示。

```
TEST_CASE("std::string supports constructing substrings ") {
  auto word = "gobbledygook"; ➊
  REQUIRE(std::string(word) == "gobbledygook"); ➋
  REQUIRE(std::string(word, 6) == "gobble"); ➌
}
```

*清单 15-2：从 C 风格字符串构造`string`*

你创建了一个名为`word`的`const char*`，指向 C 风格字符串字面量`gobbledygook` ➊。接着，你通过传入`word`来构造一个`string`。如预期的那样，结果的`string`包含`gobbledygook` ➋。在接下来的测试中，你传入数字`6`作为第二个参数。这导致`string`只取`word`的前六个字符，结果`string`包含`gobble` ➌。

此外，你还可以从其他`string`构造`string`。作为一个 STL 容器，`string`完全支持复制和移动语义。你还可以通过传入一个*子字符串*——另一个字符串的连续子集，来构造`string`。清单 15-3 展示了这三种构造方法。

```
TEST_CASE("std::string supports") {
  std::string word("catawampus"); ➊
  SECTION("copy constructing") {
    REQUIRE(std::string(word) == "catawampus"); ➋
  }
  SECTION("move constructing") {
    REQUIRE(std::string(move(word)) == "catawampus"); ➌
  }
  SECTION("constructing from substrings") {
    REQUIRE(std::string(word, 0, 3) == "cat"); ➍
    REQUIRE(std::string(word, 4) == "wampus"); ➎
  }
}
```

*清单 15-3：`string`对象的复制、移动和子字符串构造*

**注意**

*在清单 15-3 中，`word`处于一个已移动的状态，正如你从“移动语义”部分（见第 122 页）所记得的那样，这意味着它只能被重新赋值或销毁。*

在这里，你构造了一个名为 `word` 的 `string`，包含字符 `catawampus` ➊。复制构造产生了另一个 `string`，包含 `word` 的字符副本 ➋。移动构造偷取了 `word` 的字符，结果是一个包含 `catawampus` 的新 `string` ➌。最后，你可以基于子字符串构造一个新的 `string`。通过传递 `word`、起始位置为 0 和长度为 3，你构造了一个包含字符 `cat` 的新 `string` ➍。如果你改为传递 `word` 和起始位置为 4（不指定长度），你会得到从第四个字符到原始字符串末尾的所有字符，结果为 `wampus` ➎。 |

`string` 类还支持使用 `std::string_literals::operator""s` 进行字面量构造。其主要优点是符号简洁，但你也可以使用 `operator""s` 在 `string` 中轻松嵌入 null 字符，正如 示例 15-4 所示。 |

```
TEST_CASE("constructing a string with") {
  SECTION("std::string(char*) stops at embedded nulls") {
    std::string str("idioglossia\0ellohay!"); ➊
    REQUIRE(str.length() == 11); ➋
  }
  SECTION("operator\"\"s incorporates embedded nulls") {
    using namespace std::string_literals; ➌
    auto str_lit = "idioglossia\0ellohay!"s; ➍
    REQUIRE(str_lit.length() == 20); ➎
  }
}
```

*示例 15-4：构造一个 `string`*

在第一次测试中，你使用字面量 `idioglossia\0ellohay!` ➊ 构造了一个 `string`，该字符串包含 `idioglossia` ➋，由于嵌入了 null 字符，字面量的其余部分没有被复制到 `string` 中。在第二次测试中，你引入了 `std::string_literals` 命名空间 ➌，这样就可以使用 `operator""s` 从字面量直接构造一个 `string` ➍。与 `std::string` 构造函数 ➊ 不同，`operator""s` 返回一个包含整个字面量的字符串——包括嵌入的 null 字节 ➎。 |

表 15-1 总结了构造 `string` 的选项。在此表中，`c` 是 `char`，`n` 和 `pos` 是 `size_t`，`str` 是 `string` 或 C 风格字符串，`c_str` 是 C 风格字符串，`beg` 和 `end` 是输入迭代器。 |

**表 15-1：** 支持的 `std::string` 构造函数

| **构造函数** | 生成一个包含的字符串 |
| --- | --- |
| `string()` | 没有字符。 |
| `string(`n`,` c`)` | c 重复 n 次。 |
| `string(`str`,` pos`, [`n`])` | str 中从 pos 到 pos+n 的半开区间。如果省略 n，子字符串将从 pos 到 str 的末尾。 |
| `string(`c_str`, [`n`])` | c_str 的副本，长度为 n。如果 c_str 是以 null 结尾的，n 默认设置为以 null 结尾的字符串的长度。 |
| `string(`beg`,` end`)` | beg 到 end 半开区间内元素的副本。 |
| `string(`str`)` | str 的副本。 |
| `string(move(`str`))` | str 的内容，构造后处于已移动状态。 |
| `string{` c1`,` c2`,` c3 `}` | 字符 c1, c2 和 c3。 |
| `"`my string literal`"s` | 一个包含字符 `my string literal` 的字符串。 |

#### *字符串存储和小字符串优化*

和 `vector` 完全一样，`string` 使用动态存储来连续存储其组成元素。因此，`vector` 和 `string` 在复制/移动构造/赋值语义上非常相似。例如，复制操作可能比移动操作更昂贵，因为包含的元素位于动态内存中。 |

最流行的 STL 实现具有 *小字符串优化（SSO）*。如果 `string` 的内容足够小，SSO 会将其内容存储在对象的存储区内（而不是动态存储）。一般而言，少于 24 字节的 `string` 是 SSO 的候选者。实现者之所以做出此优化，是因为在许多现代程序中，大多数 `string` 都是短的。（`vector` 没有任何小优化。）

**注意**

*实际上，SSO 以两种方式影响移动操作。首先，如果 `string` 移动，任何对 `string` 元素的引用都会失效。其次，`string` 的移动操作可能比 `vector` 慢，因为 `string` 需要检查 SSO。*

一个 `string` 有一个 *大小*（或 *长度*）和一个 *容量*。大小是 `string` 中包含的字符数，而容量是 `string` 在需要调整大小之前能够容纳的字符数。

表 15-2 包含读取和操作 `string` 的大小和容量的方法。在此表中，`n` 是 `size_t` 类型。星号 (*) 表示在某些情况下，这个操作会使指向 `s` 元素的原始指针和迭代器无效。

**表 15-2：** 支持的 `std::string` 存储和长度方法

| **方法** | **返回值** |
| --- | --- |
| s`.empty()` | 如果 s 不包含任何字符，则返回 `true`；否则返回 `false`。 |
| s`.size()` | s 中字符的数量。 |
| s`.length()` | 与 s`.size()` 相同 |
| s`.max_size()` | s 的最大可能大小（由于系统/运行时的限制）。 |
| s`.capacity()` | 在需要调整大小之前，s 能够容纳的字符数量。 |
| s`.shrink_to_fit()` | `void`；发出一个非绑定请求，将 s`.capacity()` 缩减到 s`.size()`。* |
| s`.reserve([`n`])` | `void`；如果 `n >` s`.capacity()`，则调整大小以便 s 至少能容纳 n 个元素；否则，发出非绑定请求*，将 s`.capacity()` 缩减到 n 或 s`.size()`，取两者中的较大值。 |

**注意**

*截至新闻发布时，草案 C++20 标准更改了当 `reserve` 方法的参数小于 `string` 的大小时的行为。这将与 `vector` 的行为相匹配，在这种情况下没有效果，而是等同于调用 `shrink_to_fit`。*

请注意，`string` 的大小和容量方法与 `vector` 非常相似。这是由于它们存储模型的紧密性所致。

#### *元素和迭代器访问*

因为 `string` 提供对连续元素的随机访问迭代器，所以它相应地暴露了与 `vector` 类似的元素和迭代器访问方法。

为了与 C 风格的 API 进行互操作，`string` 还暴露了一个 `c_str` 方法，该方法返回一个不可修改的、以 null 结尾的字符串版本，作为 `const char*`，正如 清单 15-5 所示。

```
TEST_CASE("string's c_str method makes null-terminated strings") {
  std::string word("horripilation"); ➊
  auto as_cstr = word.c_str(); ➋
  REQUIRE(as_cstr[0] ==  'h'); ➌
  REQUIRE(as_cstr[1] ==  'o');
  REQUIRE(as_cstr[11] == 'o');
  REQUIRE(as_cstr[12] == 'n');
  REQUIRE(as_cstr[13] == '\0'); ➍
}
```

*清单 15-5：从 `string` 中提取一个 null 终止的字符串*

你构造了一个包含字符 `horripilation` ➊ 的 `string`，并使用其 `c_str` 方法提取一个名为 `as_cstr` 的 null 终止字符串 ➋。由于 `as_cstr` 是一个 `const char*`，你可以使用 `operator[]` 来说明它包含与 `word` 相同的字符 ➌，并且它是 null 终止的 ➍。

**注意**

*`std::string` 类还支持 `operator[]`，其行为与 C 风格字符串相同。*

通常，`c_str` 和 `data` 返回相同的结果，唯一的区别是 `data` 返回的引用可以是非 `const` 的。每当你操作一个 `string` 时，实施通常会确保支持 `string` 的连续内存以 null 终止符结束。列表 15-6 中的程序通过打印调用 `data` 和 `c_str` 及其地址的结果来展示这种行为。

```
#include <string>
#include <cstdio>

int main() {
  std::string word("pulchritudinous");
  printf("c_str: %s at 0x%p\n", word.c_str(), word.c_str()); ➊
  printf("data:  %s at 0x%p\n", word.data(), word.data()); ➋
}
--------------------------------------------------------------------------
c_str: pulchritudinous at 0x0000002FAE6FF8D0 ➊
data:  pulchritudinous at 0x0000002FAE6FF8D0 ➋
```

*列表 15-6：说明 `c_str` 和 `data` 返回等效地址*

`c_str` 和 `data` 返回相同的结果，因为它们指向相同的地址 ➊ ➋。由于该地址是一个 null 终止的 `string` 的起始位置，`printf` 对两次调用的输出结果相同。

表 15-3 列出了 `string` 的访问方法。注意，表中的 `n` 是 `size_t` 类型。

**表 15-3：** 支持的 `std::string` 元素和迭代器访问方法

| **方法** | **返回值** |
| --- | --- |
| s`.begin()` | 一个指向第一个元素的迭代器。 |
| s`.cbegin()` | 一个指向第一个元素的 `const` 迭代器。 |
| s`.end()` | 一个指向超出最后一个元素位置的迭代器。 |
| s`.cend()` | 一个指向超出最后一个元素位置的 `const` 迭代器。 |
| s`.at(`n`)` | 引用 s 中的第 n 个元素。如果越界，抛出 `std::out_of_range`。 |
| s`[`n`]` | 引用 s 中的第 n 个元素。如果 n `>` s`.size()`，则行为未定义。此外，s`[`s`.size()]` 必须为 0，因此写入一个非零值到该字符是未定义行为。 |
| s`.front()` | 引用第一个元素。 |
| s`.back()` | 引用最后一个元素。 |
| s`.data()` | 如果字符串非空，返回指向第一个元素的原始指针。如果字符串为空，返回指向一个 null 字符的指针。 |
| s`.c_str()` | 返回一个不可修改的、以 null 终止的 s 内容版本。 |

#### *字符串比较*

注意，`string` 支持与其他字符串以及原始 C 风格字符串的比较，使用常见的比较操作符。例如，等号 `operator==` 如果左右两侧的大小和内容相同，则返回 `true`，而不等号 `operator!=` 返回相反的结果。其余比较操作符执行 *字典顺序比较*，即按字母顺序排列，其中 *A* < *Z* < *a* < *z*，并且在其他条件相同的情况下，较短的单词小于较长的单词（例如，*pal* < *palindrome*）。列表 15-7 展示了比较的例子。

**注意**

*从技术上讲，字典顺序比较依赖于 `string` 的编码。理论上，可能存在一个系统使用默认编码，其中字母表的顺序完全混乱（例如，几乎被淘汰的 EBCDIC 编码，它将小写字母排在大写字母之前），这将影响 `string` 比较。对于与 ASCII 兼容的编码，你不需要担心，因为它们默认具有预期的字典顺序行为。*

```
TEST_CASE("std::string supports comparison with") {
  using namespace std::literals::string_literals; ➊
  std::string word("allusion"); ➋
  SECTION("operator== and !=") {
    REQUIRE(word == "allusion"); ➌
    REQUIRE(word == "allusion"s); ➍
    REQUIRE(word != "Allusion"s); ➎
    REQUIRE(word != "illusion"s); ➏
    REQUIRE_FALSE(word == "illusion"s); ➐
  }
  SECTION("operator<") {
    REQUIRE(word < "illusion"); ➑
    REQUIRE(word < "illusion"s); ➒
    REQUIRE(word > "Illusion"s); ➓
  }
}
```

*示例 15-7：`string` 类支持比较*

在这里，你引入了 `std::literals::string_literals` 命名空间，以便可以轻松地使用 `operator""s` 来构造一个 `string` ➊。你还构造了一个名为 `word` 的 `string`，其中包含字符 `allusion` ➋。在第一组测试中，你检查了 `operator==` 和 `operator!=`。

你可以看到，`word` 等于（`==`）`allusion`，无论是作为 C 风格字符串 ➌ 还是作为 `string` ➍，但是它不等于（`!=`）包含 `Allusion` ➎ 或 `illusion` ➏ 的 `string`。像往常一样，`operator==` 和 `operator!=` 总是返回相反的结果 ➐。

下一组测试使用 `operator<` 来显示 `allusion` 小于 `illusion` ➑，因为 *a* 在字典顺序上小于 *i*。比较操作适用于 C 风格字符串和 `string` ➒。示例 15-7 还显示了 `Allusion` 小于 `allusion` ➓，因为 *A* 在字典顺序上小于 *a*。

表 15-4 列出了 `string` 的比较方法。请注意，表中的 `other` 是一个 `string` 或 `char*` C 风格的字符串。

**表 15-4：** 支持的 `std::string` 比较运算符

| **方法** | **返回值** |
| --- | --- |
| s `==` other | 如果 s 和 other 具有相同的字符和长度，则返回 `true`；否则返回 `false` |
| s `!=` other | `operator==` 的相反操作 |
| s`.compare(`other`)` | 如果 s `==` other，则返回 0；如果 s `<` other，则返回负数；如果 s `>` other，则返回正数 |
| s `<` other `>` other `<=` other `>=` other | 根据字典顺序排序的相应比较操作结果 |

#### *操作元素*

对于元素操作，`string` 提供了 *许多* 方法。它支持 `vector<char>` 的所有方法，并且还有许多其他有助于处理人类语言数据的方法。

##### 添加元素

要向 `string` 中添加元素，可以使用 `push_back`，它会将一个字符插入到字符串末尾。当你想向 `string` 的末尾插入多个字符时，可以使用 `operator+=` 来追加一个字符、一个以 null 结尾的 `char*` 字符串，或一个 `string`。你也可以使用 `append` 方法，该方法有三种重载形式。首先，你可以传递一个 `string` 或一个以 null 结尾的 `char*` 字符串，以及一个可选的偏移量和一个可选的字符数来追加。其次，你可以传递一个长度和一个 `char`，它将把指定数量的 `char` 追加到字符串末尾。第三，你可以追加一个半开区间。示例 15-8 展示了所有这些操作。

```
TEST_CASE("std::string supports appending with") {
  std::string word("butt"); ➊
  SECTION("push_back") {
    word.push_back('e'); ➋
    REQUIRE(word == "butte");
  }
  SECTION("operator+=") {
    word += "erfinger"; ➌
 REQUIRE(word == "butterfinger");
  }
  SECTION("append char") {
    word.append(1, 's'); ➍
    REQUIRE(word == "butts");
  }
  SECTION("append char*") {
    word.append("stockings", 5); ➎
    REQUIRE(word == "buttstock");
  }
  SECTION("append (half-open range)") {
    std::string other("onomatopoeia"); ➏
    word.append(other.begin(), other.begin()+2); ➐
    REQUIRE(word == "button");
  }
}
```

*示例 15-8：追加到 `string`*

首先，你初始化一个名为`word`的`string`，包含字符`butt` ➊。在第一个测试中，你调用`push_back`并添加字母`e` ➋，结果是`butte`。接下来，你使用`operator+=`将`erfinger`添加到`word`中 ➌，结果是`butterfinger`。在第一次调用`append`时，你追加一个单独的`s` ➍，得到`butts`。（这个操作和`push_back`一样。）`append`的第二个重载允许你提供一个`char*`和一个长度。通过提供`stockings`和长度`5`，你将`stock`添加到`word`中，得到`buttstock` ➎。由于`append`支持半开区间，你还可以构造一个名为`other`的`string`，包含字符`onomatopoeia` ➏，并通过半开区间将前两个字符追加到`word`中，得到`button` ➐。

**注意**

*回顾“测试用例和章节”中的第 308 页，每个 Catch 单元测试的`SECTION`是独立运行的，因此对`word`的修改彼此独立：每个测试的设置代码都会重置`word`。*

##### 删除元素

要从`string`中删除元素，你有几种选择。最简单的方法是使用`pop_back`，它和`vector`一样，删除`string`中的最后一个字符。如果你想删除所有字符（从而得到一个空的`string`），可以使用`clear`方法。当你需要更精确地删除元素时，可以使用`erase`方法，它提供了多种重载方式。你可以提供一个索引和长度，删除相应的字符。你也可以提供一个迭代器来删除单个元素，或者提供一个半开区间来删除多个元素。列表 15-9 展示了如何从`string`中删除元素。

```
TEST_CASE("std::string supports removal with") {
  std::string word("therein"); ➊
  SECTION("pop_back") {
    word.pop_back();
    word.pop_back(); ➋
    REQUIRE(word == "there");
  }
 SECTION("clear") {
    word.clear(); ➌
    REQUIRE(word.empty());
  }
  SECTION("erase using half-open range") {
    word.erase(word.begin(), word.begin()+3); ➍
    REQUIRE(word == "rein");
  }
  SECTION("erase using an index and length") {
    word.erase(5, 2);
    REQUIRE(word == "there"); ➎
  }
}
```

*列表 15-9：从`string`中删除元素*

你构造一个名为`word`的`string`，包含字符`therein` ➊。在第一个测试中，你调用`pop_back`两次，首先删除字母`n`，然后删除字母`i`，因此`word`包含字符`there` ➋。接下来，你调用`clear`，这将删除`word`中的所有字符，使其变为空`string` ➌。最后两个测试使用`erase`删除`word`中某些字符的子集。在第一次使用中，你使用半开区间删除前三个字符，因此`word`包含`rein` ➍。在第二次使用中，你删除从索引 5（即`therein`中的`i`）开始，长度为两个字符的部分 ➎。像第一个测试一样，这将得到字符`there`。

##### 替换元素

要同时插入和删除元素，可以使用`string`来调用`replace`方法，它有多个重载版本。

首先，你可以提供一个半开区间和一个以空字符结尾的`char*`或`string`，然后`replace`将同时执行对半开区间内所有元素的`erase`操作，并在原区间位置插入提供的`string`。其次，你可以提供两个半开区间，`replace`将插入第二个区间，而不是`string`。

替代替换一个范围，你可以使用索引或单一的迭代器和长度。你可以提供一个新的半开范围、一个字符和大小，或一个 `string`，`replace` 将在隐式范围内替换新元素。示例 15-10 演示了这些可能性中的一些。

```
TEST_CASE("std::string replace works with") {
  std::string word("substitution"); ➊
  SECTION("a range and a char*") {
    word.replace(word.begin()+9, word.end(), "e"); ➋
    REQUIRE(word == "substitute");
  }
  SECTION("two ranges") {
    std::string other("innuendo");
    word.replace(word.begin(), word.begin()+3,
                 other.begin(), other.begin()+2); ➌
    REQUIRE(word == "institution");
  }
 SECTION("an index/length and a string") {
    std::string other("vers");
    word.replace(3, 6, other); ➍
    REQUIRE(word == "subversion");
  }
}
```

*示例 15-10：替换 `string` 的元素*

在这里，你构造了一个名为 `word` 的 `string`，其内容为 `substitution` ➊。在第一次测试中，你将从索引 9 到末尾的所有字符替换为字母 `e`，得到单词 `substitute` ➋。接下来，你将 `word` 的前三个字母替换为一个包含 `innuendo` 的 `string` 的前两个字母 ➌，得到 `institution`。最后，你使用另一种通过索引和长度来指定目标序列的方式，将字符 `stitut` 替换为字符 `vers`，得到 `subversion` ➍。

`string` 类提供了一个 `resize` 方法，用于手动设置 `string` 的长度。`resize` 方法接受两个参数：新的长度和一个可选的 `char`。如果新的 `string` 长度较小，`resize` 会忽略 `char`。如果新的 `string` 长度较大，`resize` 会按所需次数附加 `char` 以达到期望的长度。示例 15-11 展示了 `resize` 方法的使用。

```
TEST_CASE("std::string resize") {
  std::string word("shamp"); ➊
  SECTION("can remove elements") {
    word.resize(4); ➋
    REQUIRE(word == "sham");
  }
  SECTION("can add elements") {
    word.resize(7, 'o'); ➌
    REQUIRE(word == "shampoo");
  }
}
```

*示例 15-11：调整 `string` 大小*

你构造了一个名为 `word` 的 `string`，其内容为字符 `shamp` ➊。在第一次测试中，你将 `word` 调整为长度 `4`，使其包含 `sham` ➋。在第二次测试中，你将 `resize` 为长度 7，并提供可选字符 `o` 作为扩展 `word` 的值 ➌。这导致 `word` 包含 `shampoo`。

在 第 482 页 的“构造”部分，解释了一个可以提取连续字符序列并创建新 `string` 的子字符串构造函数。你还可以使用 `substr` 方法生成子字符串，该方法接受两个可选参数：一个位置参数和一个长度。位置默认值为 0（`string` 的开始），长度默认值为 `string` 的其余部分。示例 15-12 演示了如何使用 `substr`。

```
TEST_CASE("std::string substr with") {
  std::string word("hobbits"); ➊
  SECTION("no arguments copies the string") {
 REQUIRE(word.substr() == "hobbits"); ➋
  }
  SECTION("position takes the remainder") {
    REQUIRE(word.substr(3) == "bits"); ➌
  }
  SECTION("position/index takes a substring") {
    REQUIRE(word.substr(3, 3) == "bit"); ➍
  }
}
```

*示例 15-12：从 `string` 中提取子字符串*

你声明了一个名为 `word` 的 `string`，其内容为 `hobbits` ➊。如果你调用不带参数的 `sub``str`，你只是简单地复制了 `string` ➋。当你提供位置参数 `3` 时，`substr` 提取从第 3 个元素开始直到 `string` 末尾的子字符串，结果为 `bits` ➌。最后，当你提供位置（3）和长度（3）时，你将得到 `bit` ➍。

##### 字符串操作方法总结

表 15-5 列出了 `string` 的许多插入和删除方法。在此表中，`str` 是一个字符串或 C 风格的 `char*` 字符串，`p` 和 `n` 是 `size_t` 类型，ind 是 `size_t` 索引或指向 s 的迭代器，n 和 i 是 `size_t` 类型，c 是 `char`，beg 和 end 是迭代器。星号 (*) 表示此操作在某些情况下会使原始指针和迭代器失效，无法访问 `v` 的元素。 |

**表 15-5：** 支持的 `std::string` 元素操作方法 |

| **方法** | **描述** |
| --- | --- |
| s`.insert(`ind`,` str`, [`p`], [`n`])` | 将从 p 开始的 str 的 n 个元素插入到 s 中，插入位置在 ind 之前。如果没有提供 n，则插入整个 `string` 或直到 `char*` 的第一个空字符；p 默认为 0。* |
| s`.insert(`ind`,` n`,` c`)` | 在 ind 之前插入 n 个 c 的副本。* |
| s`.insert(`ind`,` beg`,` end`)` | 将从 beg 到 end 的半开区间插入到 ind 之前。* |
| s`.append(`str`, [`p`], [`n`])` | 等同于 s`.insert(`s`.end(),` str`, [`p`], [`n`])`。* |
| s`.append(`n`,` c`)` | 等同于 s`.insert(`s`.end(),` n`,` c`)`。* |
| s`.append(`beg`,` end`)` | 将从 beg 到 end 的半开区间追加到 s 的末尾。* |
| s `+=` c s `+=` str | 将 c 或 str 追加到 s 的末尾。* |
| s`.push_back(`c`)` | 将 c 添加到 s 的末尾。* |
| s`.clear()` | 移除 s 中的所有字符。* |
| s`.erase([`i`], [`n`])` | 从位置 i 开始移除 n 个字符；i 默认为 0，n 默认为 s 的剩余字符。* |
| s`.erase(`itr`)` | 删除由 itr 指向的元素。* |
| s`.erase(`beg`,` end`)` | 删除从 beg 到 end 的半开区间中的元素。* |
| s`.pop_back()` | 移除 s 的最后一个元素。* |
| s`.resize(`n`,``[`c`])`  | 调整字符串大小，使其包含 n 个字符。如果此操作增加了字符串的长度，则会添加 c 的副本，默认为 0。* |
| s`.replace(`i`,` n1`,` str`,` `[`p`], [`n2`])` | 从索引 i 开始用 str 中从 p 开始的 n2 个元素替换 n1 个字符。默认情况下，p 为 0，n2 为 str`.length()`。* |
| s`.replace(`beg`,` end`,` str`)` | 用 str 替换半开区间 beg 到 end 的元素。* |
| s`.replace(`p`,` n`,` str`)` | 用 str 从索引 p 开始到 p+n 位置替换元素。* |
| s`.replace(`beg1`,` end1`,` beg2`,` end2`)` | 用从 beg2 到 end2 的半开区间替换从 beg1 到 end1 的半开区间。* |
| s`.replace(`ind`,` c`, [`n`])` | 用 cs 从 ind 开始替换 n 个元素。* |
| s`.replace(`ind`,` beg`,` end`)` | 用半开区间 beg 到 end 替换从 ind 开始的元素。* |
| s`.substr([`p`], [`c`])` | 返回从 p 开始，长度为 c 的子字符串。默认情况下，p 为 0，c 为字符串的剩余部分。 |
| s1`.swap(`s2`)` `swap(`s1`,` s2`)` | 交换 s1 和 s2 的内容。* |

#### *搜索* |

除了前述方法，`string` 还提供了几个 *搜索方法*，它们可以帮助你找到感兴趣的子字符串和字符。每个方法执行特定类型的搜索，选择哪个方法取决于应用的具体需求。

##### find

`string` 提供的第一个方法是 `find`，它的第一个参数可以是 `string`、C 风格的 `string` 或 `char`。这个参数是你希望在 `this` 中定位的元素。你还可以选择性地提供第二个 `size_t` 类型的位置参数，告诉 `find` 从哪里开始查找。如果 `find` 未能找到子字符串，它将返回一个特殊的 `size_t` 值，即常量 `static` 成员 `std::string::npos`。示例 15-13 演示了 `find` 方法。

```
TEST_CASE("std::string find") {
  using namespace std::literals::string_literals;
  std::string word("pizzazz"); ➊
  SECTION("locates substrings from strings") {
    REQUIRE(word.find("zz"s) == 2); // pi(z)zazz ➋
  }
  SECTION("accepts a position argument") {
    REQUIRE(word.find("zz"s, 3) == 5); // pizza(z)z ➌
  }
 SECTION("locates substrings from char*") {
    REQUIRE(word.find("zaz") == 3); // piz(z)azz ➍
  }
  SECTION("returns npos when not found") {
    REQUIRE(word.find('x') == std::string::npos); ➎
  }
}
```

*示例 15-13：在 `string` 中查找子字符串*

这里，你构建了一个名为 `word` 的 `string`，其内容为 `pizzazz` ➊。在第一次测试中，你调用 `find`，并传入包含 `zz` 的 `string`，返回 `2` ➋，即 `pi``z``zazz` 中第一个 *z* 的索引。当你提供位置参数 `3`，即 `piz``z``azz` 中第二个 *z* 时，`find` 定位到第二个 *zz*，其起始位置为 `5` ➌。第三次测试中，你使用 C 风格的字符串 `zaz`，`find` 返回 3，再次对应 `piz``z``azz` 中的第二个 *z* ➍。最后，你尝试查找字符 *x*，但 `pizzazz` 中没有该字符，所以 `find` 返回 `std::string::npos` ➎。

##### rfind

`rfind` 方法是 `find` 的一种替代方法，它接受相同的参数，但以 *反向* 搜索。你可能会希望在某些情况下使用这个功能，比如，如果你在查找 `string` 末尾的特定标点符号，就如 示例 15-14 所示。

```
TEST_CASE("std::string rfind") {
  using namespace std::literals::string_literals;
  std::string word("pizzazz"); ➊
  SECTION("locates substrings from strings") {
    REQUIRE(word.rfind("zz"s) == 5); // pizza(z)z ➋
  }
  SECTION("accepts a position argument") {
    REQUIRE(word.rfind("zz"s, 3) == 2); // pi(z)zazz ➌
  }
  SECTION("locates substrings from char*") {
    REQUIRE(word.rfind("zaz") == 3); // piz(z)azz ➍
  }
  SECTION("returns npos when not found") {
    REQUIRE(word.rfind('x') == std::string::npos); ➎
  }
}
```

*示例 15-14：在 `string` 中反向查找子字符串*

使用相同的 `word` ➊，你使用与 示例 15-13 相同的参数来测试 `rfind`。给定 `zz`，`rfind` 返回 `5`，即 `pizza``z``z` 中倒数第二个 *z* ➋。当你提供位置参数 `3` 时，`rfind` 则返回 `pi``z``zazz` 中的第一个 *z* ➌。因为子字符串 `zaz` 只有一个出现，`rfind` 返回与 `find` 相同的位置 ➍。像 `find` 一样，当给定 `x` 时，`rfind` 返回 `std::string::npos` ➎。

##### find_*_of

而 `find` 和 `rfind` 用于定位 `string` 中的精确子序列，一系列相关的函数可以找到给定参数中包含的第一个字符。

`find_first_of` 函数接受一个 `string`，并定位该 `string` 中包含的第一个字符。你还可以选择性地提供一个 `size_t` 类型的位置参数，指示 `find_first_of` 从哪里开始查找。如果 `find_first_of` 未能找到匹配的字符，它将返回 `std::string::npos`。示例 15-15 演示了 `find_first_of` 函数。

```
TEST_CASE("std::string find_first_of") {
  using namespace std::literals::string_literals;
  std::string sentence("I am a Zizzer-Zazzer-Zuzz as you can plainly see."); ➊
  SECTION("locates characters within another string") {
    REQUIRE(sentence.find_first_of("Zz"s) == 7); // (Z)izzer ➋
  }
  SECTION("accepts a position argument") {
    REQUIRE(sentence.find_first_of("Zz"s, 11) == 14); // (Z)azzer ➌
  }
  SECTION("returns npos when not found") {
    REQUIRE(sentence.find_first_of("Xx"s) == std::string::npos); ➍
  }
}
```

*示例 15-15：在 `string` 中查找集合的第一个元素*

名为`sentence`的`string`包含`I am a Zizzer-Zazzer-Zuzz as you` `can plainly see.` ➊。在这里，你调用`find_first_of`并传入字符串`Zz`，它匹配小写和大写的*z*。返回值是`7`，对应于`sentence`中的第一个`Z`，即`Z``izzer` ➋。在第二个测试中，你再次传入字符串`Zz`，但同时传入位置参数`11`，对应`Zizz``e``r`中的`e`。结果是`14`，对应`Z``azzer`中的`Z` ➌。最后，你调用`find_first_of`并传入`Xx`，结果是`std::string::npos`，因为`sentence`中没有`x`（或`X`） ➍。

`string`提供了三种`find_first_of`变体：

+   `find_first_not_of`返回`string`参数中*不*包含的第一个字符。与其提供一个包含你想要查找的元素的`string`，你应该提供一个你*不*想找到的字符组成的`string`。

+   `find_last_of`执行反向匹配；与从`string`的开头或某个位置参数开始搜索并向结尾进行不同，`find_last_of`从`string`的结尾或某个位置参数开始，向开头搜索。

+   `find_last_not_of`结合了前两种变体：你传入一个不希望找到的元素组成的`string`，而`find_last_not_of`则从末尾反向搜索。

你选择的`find`函数取决于你的算法需求。你是否需要从`string`的末尾开始搜索，例如查找标点符号？如果是，使用`find_last_of`。你是否在寻找`string`中的第一个空格？如果是，使用`find_first_of`。你是否想反转搜索，查找第一个不属于某个集合的元素？那么，根据你是想从字符串的开头还是结尾开始，使用`find_first_not_of`或`find_last_not_of`。

示例 15-16 展示了这三种`find_first_of`变体。

```
TEST_CASE("std::string") {
  using namespace std::literals::string_literals;
  std::string sentence("I am a Zizzer-Zazzer-Zuzz as you can plainly see."); ➊
  SECTION("find_last_of finds last element within another string") {
    REQUIRE(sentence.find_last_of("Zz"s) == 24); // Zuz(z) ➋
  }
  SECTION("find_first_not_of finds first element not within another string") {
    REQUIRE(sentence.find_first_not_of(" -IZaeimrz"s) == 22); // Z(u)zz ➌
  }
  SECTION("find_last_not_of finds last element not within another string") {
    REQUIRE(sentence.find_last_not_of(" .es"s) == 43); // plainl(y) ➍
     }
}
```

*示例 15-16：`string`的`find_first_of`方法的替代方案*

在这里，你初始化与示例 15-15 相同的`sentence` ➊。在第一个测试中，你对`Zz`使用`find_last_of`，它从字符串的末尾反向搜索任何*z*或*Z*，返回`24`，即`Zuz``z`中的最后一个*z* ➋。接下来，你使用`find_first_not_of`并传入一堆字符（不包括字母*u*），结果是`22`，即`Z``u``zz`中第一个*u*的位置 ➌。最后，你使用`find_last_not_of`查找最后一个不等于空格、句点、*e*或*s*的字符。结果是`43`，即`plainl``y`中的*y*的位置 ➍。

##### 字符串搜索方法总结

表 15-6 列出了许多`string`的搜索方法。请注意，`s2`是一个字符串；`cstr`是一个 C 风格的`char*`字符串；`c`是一个`char`类型；`n`、`l`和`pos`是表中的`size_t`类型。

**表 15-6：** 支持的`std::string`搜索算法

| **方法** | **从** p **开始搜索并返回…的位置** |
| --- | --- |
| s`.find(`s2`, [`p`])` | 第一个子串等于 s2；p 默认为 0。 |
| s`.find(`cstr`, [`p`], [`l`])` | 第一个子串等于 cstr 的前 l 个字符；p 默认为 0；l 默认为 cstr 的长度（以空字符为终止）。 |
| s`.find(`c`, [`p`])` | 第一个字符等于 c；p 默认为 0。 |
| s`.rfind(`s2`, [`p`])` | 最后一个子串等于 s2；p 默认为`npos`。 |
| s`.rfind(`cstr`, [`p`], [`l`])` | 最后一个子串等于 cstr 的前 l 个字符；p 默认为`npos`；l 默认为 cstr 的长度（以空字符为终止）。 |
| s`.rfind(`c`, [`p`])` | 最后一个字符等于 c；p 默认为`npos`。 |
| s`.find_first_of(`s2`, [`p`])` | 第一个字符包含在 s2 中；p 默认为 0。 |
| s`.find_first_of(`cstr`, [`p`], [`l`])` | 第一个字符包含在 cstr 的前 l 个字符中；p 默认为 0；`l`默认为 cstr 的长度（以空字符为终止）。 |
| s`.find_first_of(`c`, [`p`])` | 第一个字符等于 c；p 默认为 0。 |
| s`.find_last_of(`s2`, [`p`])` | 最后一个字符包含在 s2 中；p 默认为 0。 |
| s`.find_last_of(`cstr`, [`p`], [`l`])` | 最后一个字符包含在 cstr 的前 l 个字符中；p 默认为 0；l 默认为 cstr 的长度（以空字符为终止）。 |
| s`.find_last_of(`c`, [`p`])` | 最后一个字符等于 c；p 默认为 0。 |
| s`.find_first_not_of(`s2`, [`p`])` | 第一个字符不包含在 s2 中；p 默认为 0。 |
| s`.find_first_not_of(`cstr`, [`p`], [`l`])` | 第一个字符不包含在 cstr 的前 l 个字符中；p 默认为 0；l 默认为 cstr 的长度（以空字符为终止）。 |
| s`.find_first_not_of(`c`, [`p`])` | 第一个字符不等于 c；p 默认为 0。 |
| s`.find_last_not_of(`s2`, [`p`])` | 最后一个字符不包含在 s2 中；p 默认为 0。 |
| s`.find_last_not_of(`cstr`, [`p`], [`l`])` | 最后一个字符不包含在 cstr 的前 l 个字符中；p 默认为 0；l 默认为 cstr 的长度（以空字符为终止）。 |
| s`.find_last_not_of(`c`, [`p`])` | 最后一个字符不等于 c；p 默认为 0。 |

#### *数值转换*

STL 提供了将`string`或`wstring`与基本数值类型之间进行转换的函数。给定一个数值类型，你可以使用`std::to_string`和`std::to_wstring`函数生成其`string`或`wstring`表示。这两个函数都为所有数值类型提供了重载。列表 15-17 展示了`string`和`wstring`的使用。

```
TEST_CASE("STL string conversion function") {
  using namespace std::literals::string_literals;
  SECTION("to_string") {
    REQUIRE("8675309"s == std::to_string(8675309)); ➊
  }
  SECTION("to_wstring") {
    REQUIRE(L"109951.1627776"s == std::to_wstring(109951.1627776)); ➋
  }
}
```

*列表 15-17：`string`的数字转换函数*

**注意**

*由于`double`类型本身的精度限制，第二个单元测试* ➋ *可能在你的系统上失败。*

第一个示例使用`to_string`将`int 8675309`转换为`string` ➊；第二个示例使用`to_wstring`将`double 109951.1627776`转换为`wstring` ➋。

你也可以反向转换，从 `string` 或 `wstring` 转换为数字类型。每个数字转换函数都接受一个包含字符串编码数字的 `string` 或 `wstring` 作为第一个参数。接下来，你可以提供一个可选的指向 `size_t` 的指针。如果提供了，转换函数将写入它所能转换的最后一个字符的索引（或者如果它解码了所有字符，则写入输入 `string` 的长度）。默认情况下，这个索引参数为 `nullptr`，此时转换函数不会写入索引。当目标类型是整数类型时，你可以提供第三个参数：一个 `int`，表示编码字符串的进制。这个进制参数是可选的，默认值为 10。

每个转换函数如果无法执行转换，会抛出 `std::invalid_argument`，如果转换的值超出相应类型的范围，则抛出 `std::out_of_range`。

表 15-7 列出了这些转换函数及其目标类型。在此表中，`s` 是一个字符串。如果 `p` 不是 `nullptr`，转换函数将把 `s` 中第一个未转换字符的位置写入 `p` 指向的内存中。如果所有字符都已编码，则返回 `s` 的长度。这里，`b` 是 `s` 中数字的进制表示。注意，`p` 默认为 `nullptr`，`b` 默认为 10。

**表 15-7：** `std::string` 和 `std::wstring` 的支持的数字转换函数

| **函数** | **将 `s` 转换为** |
| --- | --- |
| `stoi(`s`, [`p`], [`b`])` | 一个 `int` |
| `stol(`s`, [`p`], [`b`])` | 一个 `long` |
| `stoll(`s`, [`p`], [`b`])` | 一个 `long long` |
| `stoul(`s`, [`p`], [`b`])` | 一个 `unsigned long` |
| `stoull(`s`, [`p`], [`b`])` | 一个 `unsigned long long` |
| `stof(`s`, [`p`])` | 一个 `float` |
| `stod(`s`, [`p`])` | 一个 `double` |
| `stold(`s`, [`p`])` | 一个 `long double` |
| `to_string(`n`)` | 一个 `string` |
| `to_wstring(`n`)` | 一个 `wstring` |

示例 15-18 演示了几个数字转换函数。

```
TEST_CASE("STL string conversion function") {
  using namespace std::literals::string_literals;
  SECTION("stoi") {
    REQUIRE(std::stoi("8675309"s) == 8675309); ➊
  }
  SECTION("stoi") {
    REQUIRE_THROWS_AS(std::stoi("1099511627776"s), std::out_of_range); ➋
  }
 SECTION("stoul with all valid characters") {
    size_t last_character{};
    const auto result = std::stoul("0xD3C34C3D"s, &last_character, 16); ➌
    REQUIRE(result == 0xD3C34C3D);
    REQUIRE(last_character == 10);
  }
  SECTION("stoul") {
    size_t last_character{};
    const auto result = std::stoul("42six"s, &last_character); ➍
    REQUIRE(result == 42);
    REQUIRE(last_character == 2);
  }
  SECTION("stod") {
    REQUIRE(std::stod("2.7182818"s) == Approx(2.7182818)); ➎
  }
}
```

*示例 15-18：`string` 的字符串转换函数*

首先，使用 `stoi` 将 `8675309` 转换为整数 ➊。在第二次测试中，尝试使用 `stoi` 将 `string 1099511627776` 转换为整数。由于该值对于 `int` 来说过大，`stoi` 抛出 `std::out_of_range` ➋。接下来，使用 `stoi` 转换 `0xD3C34C3D`，但提供了两个可选参数：指向 `size_t` 的指针 `last_character` 和一个十六进制进制 ➌。`last_character` 对象的值为 `10`，即 `0xD3C34C3D` 的长度，因为 `stoi` 能解析每个字符。下一个测试中的 `string` 为 `42six`，包含无法解析的字符 `six`。当你这次调用 `stoul` 时，`result` 为 `42`，`last_character` 等于 `2`，即 `s` 中 `six` 的位置 ➍。最后，你使用 `stod` 将 `string 2.7182818` 转换为 `double` ➎。

**注意**

*Boost 的 Lexical Cast 提供了一种基于模板的替代方法，用于数值转换。有关 `boost::lexical_cast` 的文档，请参考 `<boost/lexical_cast.hpp>` 头文件中的文档。*

### 字符串视图

*字符串视图* 是一个表示常量、连续字符序列的对象。它非常类似于 `const string` 引用。实际上，字符串视图类通常实现为指向字符序列的指针和长度。

STL 提供了类模板 `std::basic_string_view`，位于 `<string_view>` 头文件中，它类似于 `std::basic_string`。模板 `std::basic_string_view` 对四种常用字符类型都有特化：

+   `char` 有 `string_view`

+   `wchar_t` 有 `wstring_view`

+   `char16_t` 有 `u16string_view`

+   `char32_t` 有 `u32string_view`

本节讨论了 `string_view` 的特化用于演示，但讨论内容同样适用于其他三种特化。

`string_view` 类支持大多数与 `string` 相同的方法；实际上，它被设计成可以替代 `const string&`。

#### *构造*

`string_view` 类支持默认构造，因此它的长度为零，并且指向 `nullptr`。重要的是，`string_view` 支持从 `const string&` 或 C 风格字符串隐式构造。你可以从 `char*` 和 `size_t` 构造 `string_view`，这样你就可以手动指定所需的长度，以便获取子串或处理嵌入的空字符。 Listing 15-19 说明了 `string_view` 的使用。

```
TEST_CASE("std::string_view supports") {
  SECTION("default construction") {
    std::string_view view; ➊
    REQUIRE(view.data() == nullptr);
    REQUIRE(view.size() == 0);
    REQUIRE(view.empty());
  }
  SECTION("construction from string") {
    std::string word("sacrosanct");
    std::string_view view(word); ➋
    REQUIRE(view == "sacrosanct");
  }
  SECTION("construction from C-string") {
    auto word = "viewership";
    std::string_view view(word); ➌
    REQUIRE(view == "viewership");
  }
  SECTION("construction from C-string and length") {
    auto word = "viewership";
    std::string_view view(word, 4); ➍
    REQUIRE(view == "view");
  }
}
```

*Listing 15-19：`string_view` 的构造函数*

默认构造的 `string_view` 指向 `nullptr`，并且是空的 ➊。当你从 `string` ➋ 或 C 风格字符串 ➌ 构造 `string_view` 时，它会指向原始内容。最后的测试提供了可选的长度参数 `4`，意味着 `string_view` 只指向前四个字符 ➍。

虽然 `string_view` 也支持复制构造和赋值，但不支持移动构造和赋值。这个设计是合理的，因为 `string_view` 不拥有它所指向的字符序列。

#### *支持的 string_view 操作*

`string_view` 类支持与 `const` `string&` 相同的许多操作，并且语义相同。以下列出了 `string` 和 `string_view` 之间共享的所有方法：

**迭代器** `begin, end`, `rbegin, rend`, `cbegin, cend`, `crbegin`, `crend`

**元素访问** `operator[], at`, `front, back`, `data`

**容量** `size, length`, `max_size`, `empty`

**搜索** `find, rfind`, `find_first_of, find_last_of`, `find_first_not_of`, `find_last_not_of`

**提取** `copy`, `substr`

**比较** `compare, operator==, operator!=` , `operator<`, `operator>`, `operator<=`, `operator>=`

除了这些共享的方法，`string_view`还支持`remove_prefix`方法，用于从`string_view`的开始位置移除指定数量的字符，以及`remove_suffix`方法，用于从末尾移除字符。列表 15-20 展示了这两种方法。

```
TEST_CASE("std::string_view is modifiable with") {
  std::string_view view("previewing"); ➊
  SECTION("remove_prefix") {
    view.remove_prefix(3); ➋
    REQUIRE(view == "viewing");
  }
  SECTION("remove_suffix") {
    view.remove_suffix(3); ➌
    REQUIRE(view == "preview");
  }
}
```

*列表 15-20: 使用`remove_prefix`和`remove_suffix`修改`string_view`*

在这里，你声明了一个`string_view`，它引用了字符串字面量`previewing` ➊。第一个测试调用`remove_prefix`，参数为`3` ➋，这将从`string_view`的前面移除三个字符，因此它现在引用`viewing`。第二个测试则调用`remove_suffix`，参数为`3` ➌，这会从`string_view`的末尾移除三个字符，结果是`preview`。

#### *所有权、使用和效率*

因为`string_view`并不拥有它所引用的序列，所以你必须确保`string_view`的生命周期是被引用序列生命周期的子集。

`string_view`最常见的用法之一是作为函数参数。当你需要与不可变的字符序列交互时，它是首选。考虑列表 15-21 中的`count_vees`函数，它用于计算字符序列中字母`v`的频率。

```
#include <string_view>

size_t count_vees(std::string_view my_view➊) {
  size_t result{};
  for(auto letter : my_view) ➋
    if (letter == 'v') result++; ➌
  return result; ➍
}
```

*列表 15-21: `count_vees` 函数*

`count_vees`函数接受一个名为`my_view`的`string_view` ➊，你使用基于范围的`for`循环 ➋遍历它。每当`my_view`中的字符等于`v`时，你就增加`result`变量 ➌，并在遍历完整个序列后返回该变量 ➍。

你可以通过简单地将`string_view`替换为`const string&`来重新实现列表 15-21，正如在列表 15-22 中所展示的那样。

```
#include <string>

size_t count_vees(const std::string& my_view) {
--snip--
}
```

*列表 15-22: 重新实现的`count_vees`函数，使用`const string&`代替`string_view`*

如果`string_view`仅仅是`const string&`的替代品，那为什么还要使用它呢？其实，如果你用`std::string`调用`count_vees`，并没有什么区别：现代编译器会生成相同的代码。

如果你用字符串字面量来调用`count_vees`，则会有很大区别：当你将字符串字面量作为`const string&`传递时，你会构造一个`string`。而当你将字符串字面量作为`string_view`传递时，你会构造一个`string_view`。构造`string`可能更昂贵，因为它可能需要分配动态内存，并且必须复制字符。而`string_view`只是一个指针和一个长度（不需要复制或分配内存）。

### 正则表达式

*正则表达式*，也叫做*regex*，是定义搜索模式的字符串。正则表达式在计算机科学中有着悠久的历史，并形成了一种用于搜索、替换和提取语言数据的迷你语言。STL 在`<regex>`头文件中提供了正则表达式的支持。

正则表达式在谨慎使用时可以非常强大、声明式且简洁；然而，也很容易写出完全无法理解的正则表达式。请有意地使用正则表达式。

#### *模式*

你使用叫做*模式*的字符串来构建正则表达式。模式使用特定的正则表达式语法来表示一个期望的字符串集，这些语法规定了构建模式的语法。换句话说，模式定义了你感兴趣的所有可能字符串的子集。STL 支持一些语法，但这里的重点是默认语法，即修改过的 ECMAScript 正则表达式语法（有关详细信息，请参见[re.grammar]）。

##### 字符类

在 ECMAScript 语法中，你将字面字符与特殊标记混合使用来描述你期望的字符串。最常见的标记可能是*字符类*，它代表一组可能的字符：`\d` 匹配任何数字，`\s` 匹配任何空白字符，`\w` 匹配任何字母数字（“单词”）字符。

表 15-8 列出了几个示例正则表达式及其可能的解释。

**表 15-8：**仅使用字符类和字面量的正则表达式模式

| **正则表达式模式** | **可能描述** |
| --- | --- |
| `\d\d\d-\d\d\d-\d\d\d\d` | 一个美国电话号码，例如 202-456-1414 |
| `\d\d:\d\d \wM` | 一个时间，格式为 HH:MM AM/PM，例如 08:49 PM |
| `\w\w\d\d\d\d\d\d` | 一个美国邮政编码，包含前置的州代码，例如 NJ07932 |
| `\w\d-\w\d` | 一个天文机械人标识符，例如 R2-D2 |
| `c\wt` | 一个以*c*开头并以*t*结尾的三字母单词，例如*cat*或*cot* |

你还可以通过将*d*、*s*或*w*大写来反转字符类，得到相反的效果：`\D`匹配任何非数字，`\S`匹配任何非空白字符，`\W`匹配任何非单词字符。

此外，你可以通过在方括号 `[]` 中显式列出它们来构建自己的字符类。例如，字符类 `[02468]` 包含偶数数字。你还可以使用连字符作为快捷方式来包含隐含的范围，因此字符类 `[0-9a-fA-F]` 包含任何十六进制数字，无论字母是否大写。最后，你可以通过在列表前加上脱字符 `^` 来反转自定义字符类。例如，字符类 `[^aeiou]` 包含所有非元音字符。

##### 量词

你可以通过使用*量词*来减少一些打字，这些量词指定左边的字符应该重复一定次数。表 15-9 列出了正则表达式量词。

**表 15-9：** 正则表达式量词

| **正则表达式量词** | **指定数量** |
| --- | --- |
| * | 0 次或更多次 |
| + | 1 次或更多次 |
| ? | 0 次或 1 次 |
| {n} | 正好 n 次 |
| {n,m} | 介于 n 和 m 之间（包括 n 和 m） |
| {n,} | 至少 n 次 |

使用量词，你可以通过模式`c\w*t`指定所有以*c*开头并以*t*结尾的单词，因为`\w*`匹配任意数量的字母数字字符。

##### 组

*组*是字符的集合。你可以通过将字符放入括号中来指定一个组。组在多个方面都有用，包括指定一个特定的集合以便最终提取和量化。

例如，你可以改进表 15-8 中的邮政编码模式，使用量词和分组，像这样：

```
(\w{2})?➊(\d{5})➋(-\d{4})?➌
```

现在你有了三个组：可选的状态➊、邮政编码➋，以及一个可选的四位数字后缀➌。正如你稍后将看到的，这些组使得从正则表达式中解析数据变得更加容易。

##### 其他特殊字符

表 15-10 列出了可用于正则表达式模式的其他特殊字符。

**表 15-10:** 示例特殊字符

| **字符** | **指定内容** |
| --- | --- |
| X&#124;Y | 字符 X 或 Y |
| \Y | 字符 Y 作为字面量（换句话说，转义它） |
| \n | 换行符 |
| \r | 回车符 |
| \t | 制表符 |
| \0 | 空字符 |
| \xYY | 对应 YY 的十六进制字符 |

#### *basic_regex*

STL 的`std::basic_regex`类模板位于`<regex>`头文件中，表示由模式构造的正则表达式。`basic_regex`类接受两个模板参数，一个是字符类型，另一个是可选的 traits 类。你几乎总是希望使用其中一种便捷的特化：`std::regex`用于`std::basic_regex<char>`，或`std::wregex`用于`std::basic_regex<wchar_t>`。

构建`regex`的主要方式是通过传递包含正则表达式模式的字符串字面量。由于模式中需要大量转义字符，尤其是反斜杠`\`，因此使用原始字符串字面量，如`R"()"`，是一个好主意。构造函数接受一个第二个可选参数，用于指定语法标志，如正则表达式语法。

虽然`regex`主要用于作为正则表达式算法的输入，但它确实提供了一些方法，允许用户与之交互。它支持常见的复制、移动构造和赋值操作，以及`swap`，还有以下功能：

+   `assign(``s``)`将模式重新分配给`s`

+   `mark_count()`返回模式中的组数

+   `flags()`返回构造时发出的语法标志

示例 15-23 展示了如何构造一个邮政编码`regex`并检查其子组。

```
#include <regex>

TEST_CASE("std::basic_regex constructs from a string literal") {
  std::regex zip_regex{ R"((\w{2})?(\d{5})(-\d{4})?)" }; ➊
  REQUIRE(zip_regex.mark_count() == 3); ➋
}
```

*示例 15-23：使用原始字符串字面量构造`regex`并提取其组数*

在这里，你使用模式`(\w{2})?(\d{5})(-\d{4})?` ➊构造了一个名为`zip_regex`的`regex`。通过使用`mark_count`方法，你会看到`zip_regex`包含三个组➋。

#### *算法*

`<regex>`类包含三种算法，用于将`std::basic_regex`应用于目标字符串：匹配、搜索或替换。你选择哪一种取决于手头的任务。

##### 匹配

*匹配* 尝试将正则表达式与 *整个* `string` 进行匹配。STL 提供了 `std::regex_match` 函数用于匹配，它有四种重载形式。

首先，你可以为 `regex_match` 提供一个 `string`、一个 C 字符串，或者一个形成半开区间的开始和结束迭代器。下一个参数是一个可选的 `std::match_results` 对象的引用，用于接收匹配的详细信息。下一个参数是定义匹配的 `std::basic_regex`，最后一个参数是一个可选的 `std::regex_constants::match_flag_type`，用于指定高级用例的附加匹配选项。`regex_match` 函数返回一个 `bool`，如果找到匹配则为 `true`，否则为 `false`。

总结来说，你可以通过以下方式调用 `regex_match`：

```
regex_match(beg, end, [mr], rgx, [flg])
regex_match(str, [mr], rgx, [flg])
```

可以提供从 `beg` 到 `end` 的半开区间，或一个 `string`/C 字符串 `str` 来进行搜索。你也可以选择提供一个名为 `mr` 的 `match_results` 来存储找到的所有匹配的详细信息。显然，你必须提供一个正则表达式 `rgx`。最后，`flg` 标志很少使用。

**注意**

*有关匹配标志 `flg` 的详细信息，请参考 [re.alg.match]。*

*子匹配*是与某个分组对应的匹配字符串的子序列。ZIP 代码匹配的正则表达式 `(\w{2})(\d{5})(-\d{4})?` 可以根据字符串产生两个或三个子匹配。例如，TX78209 包含两个子匹配 TX 和 78209，而 NJ07936-3173 包含三个子匹配 NJ、07936 和 -3173。

`match_results` 类存储零个或多个 `std::sub_match` 实例。`sub_match` 是一个简单的类模板，公开一个 `length` 方法来返回子匹配的长度，以及一个 `str` 方法来从 `sub_match` 构建一个 `string`。

有些令人困惑的是，如果 `regex_match` 成功匹配一个字符串，`match_results` 会将整个匹配字符串作为第一个元素，然后将任何子匹配存储为后续元素。

`match_results` 类提供了 表 15-11 中列出的操作。

**表 15-11：** `match_results` 的支持操作

| **操作** | **描述** |
| --- | --- |
| mr`.empty()` | 检查匹配是否成功。 |
| mr`.size()` | 返回子匹配的数量。 |
| mr`.max_size()` | 返回子匹配的最大数量。 |
| mr`.length([`i`])` | 返回子匹配 `i` 的长度，默认值为 0。 |
| mr`.position([`i`])` | 返回子匹配 `i` 的第一个位置的字符，默认值为 0。 |
| mr`.str([`i`])` | 返回表示子匹配 `i` 的字符串，默认值为 0。 |
| mr `[`i`]` | 返回一个引用，指向与子匹配 `i` 对应的 `std::sub_match` 类，默认值为 0。 |
| mr`.prefix()` | 返回一个引用，指向与匹配前序列对应的 `std::sub_match` 类。 |
| mr`.suffix()` | 返回一个引用，指向与匹配后序列对应的 `std::sub_match` 类。 |
| mr`.format(`str`)` | 返回一个 `string`，其内容按照格式字符串 str 排列。有三个特殊序列：$' 表示匹配前的字符，$' 表示匹配后的字符，$& 表示匹配的字符。 |
| mr`.begin()`mr`.end()`mr`.cbegin()`mr`.cend()` | 返回指向子匹配序列的相应迭代器。 |

`std::sub_match` 类模板有预定义的特化来与常见的字符串类型一起使用：

+   `std::csub_match` 用于 `const char*`

+   `std::wcsub_match` 用于 `const wchar_t*`

+   `std::ssub_match` 用于 `std::string`

+   `std::wssub_match` 用于 `std::wstring`

不幸的是，你将不得不手动跟踪所有这些特化，因为 `std::regex_match` 的设计。这种设计通常会让新手感到困惑，因此让我们来看一个例子。列表 15-24 使用 ZIP 代码正则表达式 `(\w{2})(\d{5})(-\d{4})?` 来匹配字符串 `NJ07936-3173` 和 `Iomega Zip 100`。

```
#include <regex>
#include <string>

TEST_CASE("std::sub_match") {
  std::regex regex{ R"((\w{2})(\d{5})(-\d{4})?)" }; ➊
  std::smatch results; ➋
  SECTION("returns true given matching string") {
    std::string zip("NJ07936-3173");
    const auto matched = std::regex_match(zip, results, regex); ➌
    REQUIRE(matched); ➍
    REQUIRE(results[0] == "NJ07936-3173"); ➎
    REQUIRE(results[1] == "NJ"); ➏
    REQUIRE(results[2] == "07936");
    REQUIRE(results[3] == "-3173");
  }
  SECTION("returns false given non-matching string") {
    std::string zip("Iomega Zip 100");
    const auto matched = std::regex_match(zip, results, regex); ➐
    REQUIRE_FALSE(matched); ➑
    }
}
```

*列表 15-24：`regex_match` 尝试将 `regex` 匹配到 `string`。*

你构造了一个带有原始文字的 `regex`：`R"((\w{2})(\d{5})(-\d{4})?)"` ➊，并默认构造了一个 `smatch` ➋。在第一次测试中，你用 `regex_match` 对有效的 ZIP 代码 `NJ07936-3173` ➌ 进行匹配，返回 `true` 值 `matched` 以表示成功 ➍。因为你为 `regex_match` 提供了一个 `smatch`，它将有效的 ZIP 代码作为第一个元素 ➎，接着是每个子组 ➏。

在第二次测试中，你使用 `regex_match` 对无效的 ZIP 代码 `Iomega Zip 100` ➐ 进行匹配，匹配失败并返回 `false` ➑。

##### 搜索

*搜索* 尝试将正则表达式匹配到字符串的 *一部分*。STL 提供了 `std::regex_search` 函数用于搜索，它本质上是 `regex_match` 的替代方案，即使只有字符串的一部分匹配 `regex`，它也会成功。

例如，字符串 `The string NJ07936-3173 is a ZIP Code.` 包含一个 ZIP 代码。但使用 `std::regex_match` 对其应用 ZIP 正则表达式将返回 `false`，因为 `regex` 没有匹配到 *整个* 字符串。然而，使用 `std::regex_search` 会返回 `true`，因为字符串中嵌入了有效的 ZIP 代码。列表 15-25 演示了 `regex_match` 和 `regex_search` 的使用。

```
TEST_CASE("when only part of a string matches a regex, std::regex_ ") {
  std::regex regex{ R"((\w{2})(\d{5})(-\d{4})?)" }; ➊
  std::string sentence("The string NJ07936-3173 is a ZIP Code."); ➋
  SECTION("match returns false") {
    REQUIRE_FALSE(std::regex_match(sentence, regex)); ➌
  }
  SECTION("search returns true") {
    REQUIRE(std::regex_search(sentence, regex)); ➍
  }
}
```

*列表 15-25：比较 `regex_match` 和 `regex_search`*

如前所述，你构造了 ZIP `regex` ➊。你还构造了示例字符串 `sentence`，其中嵌入了有效的 ZIP 代码 ➋。第一个测试使用 `regex_match` 对 `sentence` 和 `regex` 进行匹配，返回 `false` ➌。第二个测试则调用 `regex_search`，使用相同的参数，返回 `true` ➍。

##### 替换

*替换* 将正则表达式匹配的内容替换为替换文本。STL 提供了 `std::regex_replace` 函数来进行替换。

在最基本的用法中，你传递给 `regex_replace` 三个参数：

+   一个源 `string`/C-string/半开区间进行搜索

+   一个正则表达式

+   一个替换字符串

例如，示例 15-26 将短语 `queueing and cooeeing in eutopia` 中的所有元音字母替换为下划线（`_`）。

```
TEST_CASE("std::regex_replace") {
  std::regex regex{ "[aeoiu]" }; ➊
  std::string phrase("queueing and cooeeing in eutopia"); ➋
  const auto result = std::regex_replace(phrase, regex, "_"); ➌
  REQUIRE(result == "q_____ng _nd c_____ng _n __t_p__"); ➍
}
```

*示例 15-26：使用 `std::regex_replace` 将元音字母替换为下划线*

你构造一个包含所有元音字母集合 ➊ 的 `std::regex`，并且创建一个名为 `phrase` 的 `string`，其中包含元音丰富的内容 `queueing and cooeeing in eutopia` ➋。接着，你调用 `std::regex_replace`，传入 `phrase`、正则表达式和字符串字面量 `_` ➌，它将所有元音字母替换为下划线 ➍。

**注意**

*Boost Regex 提供了与 STL 在 `<boost/regex.hpp>` 头文件中的正则表达式支持相对应的功能。另一个 Boost 库，Xpressive，提供了一种替代方法，可以直接在 C++ 代码中表达正则表达式。它具有一些主要优点，如表达能力和编译时语法检查，但其语法不可避免地与标准的正则表达式语法（如 POSIX、Perl 和 ECMAScript）有所不同。*

### Boost 字符串算法

Boost 的字符串算法库提供了丰富的 `string` 操作函数。它包含了常见的字符串处理任务的函数，例如修剪、大小写转换、查找/替换和评估特征。你可以在 `boost::algorithm` 命名空间和 `<boost/algorithm/string.hpp>` 便捷头文件中访问所有 Boost 字符串算法函数。

#### *Boost Range*

*范围*是一个概念（在第六章编译时多态性的意义上），它有一个起点和终点，允许你遍历其中的元素。范围旨在改进传递半开范围作为一对迭代器的做法。通过将这对迭代器替换为一个单一对象，你可以*组合*算法，通过使用一个算法的范围结果作为另一个算法的输入。例如，如果你想将一系列字符串转换为全大写并对它们进行排序，你可以将一个操作的结果直接传递给另一个。这种操作单独使用迭代器通常是无法做到的。

范围目前还不是 C++ 标准的一部分，但已有多个实验性实现。其中一个实现是 Boost Range，并且由于 Boost 字符串算法广泛使用 Boost Range，现在我们来了解一下它。

Boost Range 概念类似于 STL 容器概念。它提供了常见的 `begin`/`end` 方法，用于暴露范围内元素的迭代器。每个范围都有一个*遍历类别*，它指示范围支持的操作：

+   一个*单向范围*允许一次性、正向迭代。

+   一个*正向范围*允许（无限次）正向迭代，并满足单向范围的要求。

+   一个*双向范围*允许正向和反向迭代，并满足正向范围的要求。

+   一个*随机访问范围*允许任意元素访问，并满足双向范围的要求。

Boost 字符串算法是为 `std::string` 设计的，它满足随机访问范围的概念。在大多数情况下，Boost 字符串算法接受 Boost Range 而不是 `std::string` 对用户来说是完全透明的抽象。在阅读文档时，你可以将 `Range` 心理上替换为 `string`。

#### *谓词*

Boost 字符串算法广泛地集成了谓词。你可以通过引入 `<boost/algorithm/string/predicate.hpp>` 头文件直接使用它们。这个头文件中的大多数谓词接受两个范围 `r1` 和 `r2`，并根据它们之间的关系返回 `bool`。例如，谓词 `starts_with` 如果 `r1` 以 `r2` 开头，则返回 `true`。

每个谓词都有一个不区分大小写的版本，你可以通过在方法名前加字母 `i` 来使用，如 `istarts_with`。列表 15-27 演示了 `starts_with` 和 `istarts_with`。

```
#include <string>
#include <boost/algorithm/string/predicate.hpp>

TEST_CASE("boost::algorithm") {
  using namespace boost::algorithm;
  using namespace std::literals::string_literals;
  std::string word("cymotrichous"); ➊
  SECTION("starts_with tests a string's beginning") {
    REQUIRE(starts_with(word, "cymo"s)); ➋
  }
  SECTION("istarts_with is case insensitive") {
    REQUIRE(istarts_with(word, "cYmO"s)); ➌
  }
}
```

*列表 15-27：`starts_with` 和 `istarts_with` 都检查范围的起始字符。*

你初始化一个包含 `cymotrichous` 的 `string` ➊。第一次测试显示，当使用 `word` 和 `cymo` ➋ 时，`starts_with` 返回 `true`。不区分大小写的版本 `istarts_with` 在使用 `word` 和 `cYmO` ➌ 时也返回 `true`。

请注意，`<boost/algorithm/string/predicate.hpp>` 还包含一个 `all` 谓词，它接受一个范围 `r` 和一个谓词 `p`。如果 `p` 对 `r` 中的所有元素计算结果为 `true`，则返回 `true`，正如 列表 15-28 所示。

```
TEST_CASE("boost::algorithm::all evaluates a predicate for all elements") {
  using namespace boost::algorithm;
  std::string word("juju"); ➊
  REQUIRE(all(word➋, [](auto c) { return c == 'j' || c =='u'; }➌));
}
```

*列表 15-28：`all` 谓词评估范围内所有元素是否满足谓词。*

你初始化一个包含 `juju` 的字符串 ➊，并将其作为范围 ➋ 传递给 `all`。你传递一个 lambda 谓词，它对字母 `j` 和 `u` 返回 `true` ➌。因为 `juju` 只包含这些字母，`all` 返回 `true`。

表 15-12 列出了 `<boost/algorithm/string/predicate.hpp>` 中可用的谓词。在此表中，`r, r1` 和 `r2` 是字符串范围，`p` 是元素比较谓词。

**表 15-12：** Boost 字符串算法库中的谓词

| **谓词** | **返回** `true` **如果** |
| --- | --- |
| `starts_with(`r1`,` r2`, [`p`])``istarts_with(`r1`,` r2`)` | r1 以 r2 开头；p 用于逐字符比较。 |
| `ends_with(`r1`,` r2`, [`p`])``iends_with(`r1`,` r2`)` | r1 以 r2 结尾；p 用于逐字符比较。 |
| `contains(`r1`,` r2`, [`p`])``icontains(`r1`,` r2`)` | r1 包含 r2；p 用于逐字符比较。 |
| `equals(`r1`,` r2`, [`p`])``iequals(`r1`,` r2) | r1 等于 r2；p 用于逐字符比较。 |
| `lexicographical_compare(`r1`,` r2`, [`p`])``ilexicographical_compare(`r1`,` r2) | r1 在字典顺序上小于 r2；p 用于逐字符比较。 |
| `all(`r`, [`p`])` | r 的所有元素对于 p 返回 `true`。 |

以 `i` 开头的函数变种是不区分大小写的。

#### *分类器*

*分类器* 是评估字符某些特征的谓词。`<boost/algorithm/string/classification.hpp>` 头文件提供了用于创建分类器的生成器。*生成器* 是一种非成员函数，类似于构造函数。一些生成器接受参数，以自定义分类器。

**注意**

*当然，你也可以像使用自己定义的函数对象（比如 lambda）一样，轻松地创建你自己的谓词，但 Boost 为了方便提供了一些现成的分类器。*

`is_alnum` 生成器，例如，用于创建一个分类器来判断一个字符是否为字母数字字符。示例 15-29 说明了如何独立使用这个分类器或与 `all` 一起使用。

```
#include <boost/algorithm/string/classification.hpp>

TEST_CASE("boost::algorithm::is_alnum") {
  using namespace boost::algorithm;
  const auto classifier = is_alnum(); ➊
  SECTION("evaluates alphanumeric characters") {
    REQUIRE(classifier('a')); ➋
    REQUIRE_FALSE(classifier('$')); ➌
  }
  SECTION("works with all") {
    REQUIRE(all("nostarch", classifier)); ➍
    REQUIRE_FALSE(all("@nostarch", classifier)); ➎
  }
}
```

*示例 15-29：`is_alum` 生成器判断字符是否为字母数字。*

在这里，你从 `is_alnum` 生成器构造一个 `classifier` ➊。第一个测试使用 `classifier` 来评估字符 `a` 是否为字母数字 ➋，而 `$` 则不是 ➌。由于所有分类器都是作用于字符的谓词，你可以将它们与前一节讨论的 `all` 谓词结合使用，以确定 `nostarch` 是否包含所有字母数字字符 ➍，而 `@nostarch` 则不包含 ➎。

表 15-13 列出了 `<boost/algorithm/string/classification.hpp>` 中可用的字符分类。在这个表中，`r` 是一个字符串范围，`beg` 和 `end` 是元素比较谓词。

**表 15-13：** Boost 字符串算法库中的字符谓词

| **谓词** | **当元素是 . . . 时返回** true |
| --- | --- |
| `is_space` | 空格 |
| `is_alnum` | 字母数字字符 |
| `is_alpha` | 字母字符 |
| `is_cntrl` | 控制字符 |
| `is_digit` | 十进制数字 |
| `is_graph` | 图形字符 |
| `is_lower` | 小写字母 |
| `is_print` | 可打印字符 |
| `is_punct` | 标点符号字符 |
| `is_upper` | 大写字母 |
| `is_xdigit` | 十六进制数字 |
| `is_any_of(`r`)` | 包含在 r 中 |
| `is_from_range(`beg`,` end`)` | 包含在从 beg 到 end 的范围内 |

#### *查找器*

*查找器* 是一个概念，用来确定范围内与某些特定条件（通常是谓词或正则表达式）匹配的元素位置。Boost 字符串算法库在 `<boost/algorithm/string/finder.hpp>` 头文件中提供了一些生成器，用于生成查找器。

例如，`nth_finder` 生成器接受一个范围 `r` 和一个索引 `n`，它创建一个查找器，搜索一个范围（由 `begin` 和 `end` 迭代器表示），查找 `r` 的第 `n` 次出现，如 示例 15-30 所示。

```
#include <boost/algorithm/string/finder.hpp>

TEST_CASE("boost::algorithm::nth_finder finds the nth occurrence") {
  const auto finder = boost::algorithm::nth_finder("na", 1); ➊
  std::string name("Carl Brutananadilewski"); ➋
  const auto result = finder(name.begin(), name.end()); ➌
  REQUIRE(result.begin() == name.begin() + 12); ➍ // Brutana(n)adilewski
  REQUIRE(result.end() == name.begin() + 14); ➎ // Brutanana(d)ilewski
}
```

*示例 15-30：`nth_finder` 生成器创建一个查找器，用于定位一个序列的第 *n* 次出现。*

你可以使用 `nth_finder` 生成器来创建 `finder`，它会定位范围内 `na` 的第二个实例（`n` 是从零开始的） ➊。接下来，你构造一个包含 `Carl Brutananadilewski` 的 `name` ➋，并使用 `name` 的 `begin` 和 `end` 迭代器调用 `finder` ➌。`result` 是一个范围，其 `begin` 指向 `Brutana``n``adilewski` 中第二个 *n* ➍，而 `end` 指向 `Brutanana``d``ilewski` 中第一个 *d* ➎。

表 15-14 列出了 `<boost/algorithm/string/finder.hpp>` 中可用的查找器。在此表中，`s` 是字符串，`p` 是元素比较谓词，`n` 是整数值，`beg` 和 `end` 是迭代器，`rgx` 是正则表达式，`r` 是字符串范围。

**表 15-14：** Boost 字符串算法库中的查找器

| **生成器** | **创建一个查找器，当被调用时返回...** |
| --- | --- |
| `first_finder(`s`,` p`)` | 使用 p 查找匹配 s 的第一个元素 |
| `last_finder(`s`,` p)` | 使用 p 查找匹配 s 的最后一个元素 |
| `nth_finder(`s`,` p`,` n`)` | 使用 p 查找匹配 s 的第 n 个元素 |
| `head_finder(`n`)` | 前 n 个元素 |
| `tail_finder(`n`)` | 后 n 个元素 |
| `token_finder(`p`)` | 匹配 p 的字符 |
| `range_finder(`r`)``range_finder(`beg`,` end`)` | 不考虑输入，始终返回 r |
| `regex_finder(`rgx`)` | 匹配 rgx 的第一个子字符串 |

**注意**

*Boost 字符串算法指定了一个格式化器概念，它将查找器的结果呈现给替换算法。只有高级用户才需要这些算法。更多信息，请参考 `<boost/algorithm/string/find_format.hpp>` 头文件中的 `find_format` 算法文档。*

#### *修改算法*

Boost 包含了许多用于修改 `string`（范围）的算法。在 `<boost/algorithm/string/case_conv.hpp>`、`<boost/algorithm/string/trim.hpp>` 和 `<boost/algorithm/string/replace.hpp>` 头文件中，存在将大小写转换、修剪、替换和删除多种不同方式的算法。

例如，`to_upper` 函数将把字符串中的所有字母转换为大写。如果你想保持原始字符串不变，可以使用 `to_upper_copy` 函数，它会返回一个新的对象。示例 15-31 说明了 `to_upper` 和 `to_upper_copy`。

```
#include <boost/algorithm/string/case_conv.hpp>

TEST_CASE("boost::algorithm::to_upper") {
  std::string powers("difficulty controlling the volume of my voice"); ➊
  SECTION("upper-cases a string") {
    boost::algorithm::to_upper(powers); ➋
    REQUIRE(powers == "DIFFICULTY CONTROLLING THE VOLUME OF MY VOICE"); ➌
  }
  SECTION("_copy leaves the original unmodified") {
    auto result = boost::algorithm::to_upper_copy(powers); ➍
    REQUIRE(powers == "difficulty controlling the volume of my voice"); ➎
    REQUIRE(result == "DIFFICULTY CONTROLLING THE VOLUME OF MY VOICE"); ➏
  }
}
```

*示例 15-31：`to_upper` 和 `to_upper_copy` 都会将 `string` 的字母转换为大写。*

你创建了一个名为 `powers` 的 `string` ➊。第一次测试调用 `to_upper` 函数作用于 `powers` ➋，它会原地修改 `powers`，使其包含所有大写字母 ➌。第二次测试使用 `_copy` 变体，创建一个名为 `result` 的新 `string` ➍。此时，`powers` 字符串不受影响 ➎，而 `result` 包含一个全大写的版本 ➏。

一些 Boost 字符串算法，例如 `replace_first`，也有不区分大小写的版本。只需在前面加上 `i`，匹配将不受大小写限制。对于像 `replace_first` 这样的算法，它们还有 `_copy` 变种，任何排列组合都能正常工作（`replace_first`、`ireplace_first`、`replace_first_copy` 和 `ireplace_first_copy`）。

`replace_first` 算法及其变种接受输入范围 `s`、匹配范围 `m` 和替换范围 `r`，并将 `s` 中第一个匹配 `m` 的实例替换为 `r`。列表 15-32 说明了 `replace_first` 和 `i_replace_first`。

```
#include <boost/algorithm/string/replace.hpp>

TEST_CASE("boost::algorithm::replace_first") {
  using namespace boost::algorithm;
  std::string publisher("No Starch Press"); ➊
  SECTION("replaces the first occurrence of a string") {
    replace_first(publisher, "No", "Medium"); ➋
    REQUIRE(publisher == "Medium Starch Press"); ➌
  }
  SECTION("has a case-insensitive variant") {
    auto result = ireplace_first_copy(publisher, "NO", "MEDIUM"); ➍
    REQUIRE(publisher == "No Starch Press"); ➎
    REQUIRE(result == "MEDIUM Starch Press"); ➏
  }}
```

*列表 15-32：`replace_first` 和 `i_replace_first` 都会替换匹配的 `string` 序列。*

在这里，你构造了一个名为 `publisher` 的 `string`，其值为 `No Starch Press` ➊。第一个测试调用 `replace_first`，以 `publisher` 作为输入字符串，`No` 作为匹配字符串，`Medium` 作为替换字符串 ➋。随后，`publisher` 的值变为 `Medium Starch Press` ➌。第二个测试使用不区分大小写并执行复制的 `ireplace_first_copy` 变种。你分别将 `NO` 和 `MEDIUM` 作为匹配和替换字符串 ➍，此时 `result` 包含 `MEDIUM Starch Press` ➏，而 `publisher` 不受影响 ➎。

表 15-15 列出了 Boost 字符串算法中许多可用的修改算法。在这个表格中，`r`、`s`、`s1` 和 `s2` 是字符串；`p` 是元素比较谓词；`n` 是整数值；`rgx` 是正则表达式。

**表 15-15：** Boost 字符串算法库中的修改算法

| **算法** | **描述** |
| --- | --- |
| `to_upper(`s`)``to_upper_copy(`s`)` | 将 s 转换为全大写 |
| `to_lower(`s`)``to_lower_copy(`s`)` | 将 s 转换为全小写 |
| `trim_left_copy_if(`s`, [`p`])``trim_left_if(`s`, [`p`])``trim_left_copy(`s`)``trim_left(`s`)` | 移除 s 中的前导空格 |
| `trim_right_copy_if(`s`, [`p`])``trim_right_if(`s`, [`p`])``trim_right_copy(`s`)``trim_right(`s`)` | 移除 s 中的尾随空格 |
| `trim_copy_if(`s`, [`p`])``trim_if(`s`, [`p`])``trim_copy(`s`)``trim(`s`)` | 移除 s 中的前导和尾随空格 |
| `replace_first(`s1`,` s2`,` r`)``replace_first_copy(`s1`,` s2`,` r`)``ireplace_first(`s1`,` s2`,` r`)``ireplace_first_copy(`s1`,` s2`,` r`)` | 将 s1 中第一个出现的 s2 替换为 r |
| `erase_first(`s1`,` s2`)``erase_first_copy(`s1`,` s2`)``ierase_first(`s1`,` s2`)``ierase_first_copy(`s1`,` s2`)` | 删除 s1 中第一个出现的 s2 |
| `replace_last(`s1`,` s2`,` r`)``replace_last_copy(`s1`,` s2`,` r`)``ireplace_last(`s1`,` s2`,` r`)``ireplace_last_copy(`s1`,` s2`,` r`)` | 将 s1 中最后一个出现的 s2 替换为 r |
| `erase_last(`s1`,` s2`)``erase_last_copy(`s1`,` s2`)``ierase_last(`s1`,` s2`)``ierase_last_copy(`s1`,` s2`)` | 删除 s1 中最后一个出现的 s2 |
| `replace_nth(`s1`,` s2`,` n`,` r`)``replace_nth_copy(`s1`,` s2`,` n`,` r`)``ireplace_nth(`s1`,` s2`,` n`,` r`)``ireplace_nth_copy(`s1`,` s2`,` n`,` r`)` | 替换 s1 中第 n 次出现的 s2 为 r |
| `erase_nth(`s1`,` s2`,` n`)``erase_nth_copy(`s1`,` s2`,` n`)``ierase_nth(`s1`,` s2`,` n`)``ierase_nth_copy(`s1`,` s2`,` n`)` | 删除 s1 中第 n 次出现的 s2 |
| `replace_all(`s1`,` s2`,` r`)``replace_all_copy(`s1`,` s2`,` r`)``ireplace_all(`s1`,` s2`,` r`)``ireplace_all_copy(`s1`,` s2`,` r`)` | 用 r 替换 s1 中所有 s2 的出现 |
| `erase_all(`s1`,` s2`)``erase_all_copy(`s1`,` s2`)``ierase_all(`s1`,` s2`)``ierase_all_copy(`s1`,` s2`)` | 删除 s1 中所有 s2 的出现 |
| `replace_head(`s`,` n`,` r`)``replace_head_copy(`s`,` n`,` r`)` | 用 r 替换 s 的前 n 个字符 |
| `erase_head(`s`,` n`)``erase_head_copy(`s`,` n`)` | 删除 s 的前 n 个字符 |
| `replace_tail(`s`,` n`,` r`)``replace_tail_copy(`s`,` n`,` r`)` | 用 r 替换 s 的最后 n 个字符 |
| `erase_tail(`s`,` n`)``erase_tail_copy(`s`,` n`)` | 删除 s 的最后 n 个字符 |
| `replace_regex(`s`,` rgx`,` r`)``replace_regex_copy(`s`,` rgx`,` r`)` | 替换 s 中 rgx 的第一次出现为 r |
| `erase_regex(`s`,` rgx`)``erase_regex_copy(`s`,` rgx`)` | 删除 s 中 rgx 的第一次出现 |
| `replace_all_regex(`s`,` rgx`,` r`)``replace_all_regex_copy(`s`,` rgx`,` r`)` | 替换 s 中所有 rgx 的实例为 r |
| `erase_all_regex(`s`,` rgx`)``erase_all_regex_copy(`s`,` rgx`)` | 删除 s 中所有 rgx 的实例 |

#### *拆分与连接*

Boost 字符串算法包含用于拆分和连接字符串的函数，分别位于 `<boost/algorithm/string/split.hpp>` 和 `<boost/algorithm/string/join.hpp>` 头文件中。

要拆分一个 `string`，你需要提供 `split` 函数一个 STL 容器 `res`、一个范围 `s` 和一个谓词 `p`。它将使用谓词 `p` 来确定分隔符，并将结果插入到 `res` 中。 列表 15-33 演示了 `split` 函数。

```
#include <vector>
#include <boost/algorithm/string/split.hpp>
#include <boost/algorithm/string/classification.hpp>

TEST_CASE("boost::algorithm::split splits a range based on a predicate") {
  using namespace boost::algorithm;
  std::string publisher("No Starch Press"); ➊
  std::vector<std::string> tokens; ➋
  split(tokens, publisher, is_space()); ➌
  REQUIRE(tokens[0] == "No"); ➍
  REQUIRE(tokens[1] == "Starch");
  REQUIRE(tokens[2] == "Press");
}
```

*列表 15-33：`split` 函数将一个 `string` 拆分成多个标记。*

再次使用 `publisher` ➊，你创建一个名为 `tokens` 的 `vector` 来存储结果 ➋。你调用 `split`，将 `tokens` 作为结果容器，`publisher` 作为范围，`is_space` 作为你的谓词 ➌。这将把 publisher 按空格拆分。之后，`tokens` 包含 `No, Starch` 和 `Press`，正如预期的那样 ➍。

你可以使用 `join` 执行逆操作，它接受一个 STL 容器 `seq` 和一个分隔符字符串 `sep`。`join` 函数会将 `seq` 中的每个元素与 `sep` 分隔符连接在一起。

列表 15-34 演示了 `join` 函数的实用性以及牛津逗号的不可或缺性。

```
#include <vector>
#include <boost/algorithm/string/join.hpp>

TEST_CASE("boost::algorithm::join staples tokens together") {
  std::vector<std::string> tokens{ "We invited the strippers",
                                   "JFK", "and Stalin." }; ➊
  auto result = boost::algorithm::join(tokens, ", "); ➋
  REQUIRE(result == "We invited the strippers, JFK, and Stalin."); ➌
}
```

*列表 15-34：`join` 函数将 `string` 标记与分隔符连接在一起。*

你实例化了一个名为 `tokens` 的 `vector`，包含三个 `string` 对象 ➊。接着，你使用 `join` 将 token 的构成元素用逗号和空格连接在一起 ➋。结果是一个单一的 `string`，其中包含了通过逗号和空格连接的构成元素 ➌。

表 15-16 列出了 `<boost/algorithm/string/split.hpp>` 和 `<boost/algorithm/string/join.hpp>` 中提供的许多拆分/连接算法。在此表中，`res, s, s1`, `s2` 和 `sep` 是字符串；`seq` 是字符串的范围；`p` 是元素比较谓词；`rgx` 是正则表达式。

**表 15-16：** Boost 字符串算法库中的 `split` 和 `join` 算法

| **函数** | **描述** |
| --- | --- |
| `find_all(`res`,` s1`,` s2`)``ifind_all(`res`,` s1`,` s2`)``find_all_regex(`res`,` s1`,` rgx`)``iter_find(`res`,` s1`,` s2`)` | 查找 s1 中所有出现的 s2 或 rgx，将每个结果写入 res |
| `split(`res`,` s`,` p`)``split_regex(`res`,` s`,` rgx`)``iter_split(`res`,` s`,` s2`)` | 使用 p、rgx 或 s2 拆分 s，并将结果写入 res |
| `join(`seq`,` sep`)` | 返回一个 `string`，使用 sep 作为分隔符连接 seq 中的元素 |
| `join_if(`seq`,` sep`,` p`)` | 返回一个 `string`，连接 seq 中所有匹配 p 的元素，并使用 sep 作为分隔符 |

#### *查找*

Boost 字符串算法在 `<boost/algorithm/string/find.hpp>` 头文件中提供了许多查找范围的函数。这些函数本质上是 表 15-8 中查找器的便捷封装。

例如，`find_head` 函数接受一个范围 `s` 和一个长度 `n`，并返回一个包含 `s` 的前 `n` 个元素的范围。示例 15-35 演示了 `find_head` 函数的用法。

```
#include <boost/algorithm/string/find.hpp>

TEST_CASE("boost::algorithm::find_head computes the head") {
  std::string word("blandishment"); ➊
  const auto result = boost::algorithm::find_head(word, 5); ➋
  REQUIRE(result.begin() == word.begin()); ➌ // (b)landishment
  REQUIRE(result.end() == word.begin()+5); ➍ // bland(i)shment
}
```

*示例 15-35：`find_head` 函数从 `string` 的开头创建一个范围。*

你构建了一个名为 `word` 的 `string`，其中包含 `blandishment` ➊。然后，你将它和长度参数 `5` 一起传递给 `find_head` ➋。`result` 的 `begin` 指向 `word` 的开始位置 ➌，`end` 指向第五个元素之后的位置 ➍。

表 15-17 列出了 `<boost/algorithm/string/find.hpp>` 中提供的许多查找算法。在此表中，`s, s1` 和 `s2` 是字符串；`p` 是元素比较谓词；`rgx` 是正则表达式；`n` 是一个整数值。

**表 15-17：** Boost 字符串算法库中的查找算法

| **谓词** | **查找 . . .** |
| --- | --- |
| `find_first(`s1`,` s2`)``ifind_first(`s1`,` s2`)` | s1 中首次出现 s2 的位置 |
| `find_last(`s1`,` s2`)``ifind_last(`s1`,` s2`)` | s1 中最后一次出现 s2 的位置 |
| `find_nth(`s1`,` s2`,` n`)``ifind_nth(`s1`,` s2`,` n`)` | s1 中第 n 次出现 s2 的位置 |
| `find_head(`s`,` n`)` | s 的前 n 个字符 |
| `find_tail(`s`,` n`)` | s 的最后 n 个字符 |
| `find_token(`s`,` p`)` | s 中第一个与 p 匹配的字符 |
| `find_regex(`s`,` rgx`)` | s 中与 rgx 匹配的第一个子字符串 |
| `find(`s`,` fnd`)` | 将 fnd 应用于 s 的结果 |

### Boost Tokenizer

Boost Tokenizer 的 `boost::tokenizer` 是一个类模板，它提供了一个 `string` 中包含的标记序列的视图。一个 `tokenizer` 接受三个可选的模板参数：一个 tokenizer 函数，一个迭代器类型，和一个字符串类型。

*tokenizer 函数* 是一个谓词，用来判断一个字符是否是分隔符（返回 `true`）或不是（返回 `false`）。默认的 tokenizer 函数将空格和标点符号视为分隔符。如果你想明确指定分隔符，可以使用 `boost::char_separator<char>` 类，它接受一个包含所有分隔符字符的 C 字符串。例如，`boost::char_separator<char>(";|,")` 会在分号（`;`）、管道符号（`|`）和逗号（`,`）处分割。

迭代器类型和字符串类型与你想要分割的 `string` 类型对应。默认情况下，它们分别是 `std::string::const_iterator` 和 `std::string`。

因为 `tokenizer` 不会分配内存，而 `boost::algorithm::split` 会，因此当你只需要一次迭代 `string` 的标记时，强烈建议使用前者。

`tokenizer` 提供了 `begin` 和 `end` 方法，它们返回输入迭代器，因此你可以将其视为一个与底层标记序列对应的值范围。

清单 15-36 按逗号分割标志性回文 `A man, a plan, a canal, Panama!`。

```
#include<boost/tokenizer.hpp>
#include<string>

TEST_CASE("boost::tokenizer splits token-delimited strings") {
  std::string palindrome("A man, a plan, a canal, Panama!"); ➊
  boost::char_separator<char> comma{ "," }; ➋
  boost::tokenizer<boost::char_separator<char>> tokens{ palindrome, comma }; ➌
  auto itr = tokens.begin(); ➍
 REQUIRE(*itr == "A man"); ➎
  itr++; ➏
  REQUIRE(*itr == " a plan");
  itr++;
  REQUIRE(*itr == " a canal");
  itr++;
  REQUIRE(*itr == " Panama!");
}
```

*清单 15-36：`boost::tokenizer` 按指定分隔符分割字符串。*

在这里，你构建了 `palindrome` ➊，`char_separator` ➋ 和相应的 `tokenizer` ➌。接下来，你使用其 `begin` 方法 ➍ 从 tokenizer 中提取一个迭代器。你可以像通常那样处理结果迭代器，解引用其值 ➎ 并递增到下一个元素 ➏。

### 本地化

*locale* 是一个用于编码文化偏好的类。locale 概念通常被编码在你的应用程序运行的操作环境中。它还控制许多偏好设置，例如字符串比较；日期和时间、货币和数字格式；邮政编码和 ZIP 代码；以及电话号码。

STL 提供了 `std::locale` 类以及 `<locale>` 头文件中的许多辅助函数和类。

由于简洁性（并且部分原因是本书的主要读者是讲英语的人），本章将不再深入探讨 locales。

### 总结

本章详细介绍了`std::string`及其生态系统。你在探索它与`std::vector`的相似性后，学习了它处理人类语言数据的内建方法，例如比较、添加、删除、替换和搜索。你了解了数字转换函数如何让你在数字和字符串之间转换，并且分析了`std::string_view`在传递字符串时的作用。你还学习了如何利用正则表达式执行基于复杂模式的匹配、搜索和替换。最后，你深入了解了 Boost 字符串算法库，它补充并扩展了`std::string`的内建方法，提供了额外的搜索、替换、修剪、删除、分割和连接方法。

**练习**

**15-1.** 重构清单 9-30 和 9-31 中的直方图计算器，改用`std::string`。根据程序的输入构造一个`string`，并修改`AlphaHistogram`的`ingest`方法，使其接受`string_view`或`const string&`。使用基于范围的`for`循环遍历已输入的`string`元素。将`counts`字段的类型替换为关联容器。

**15-2.** 实现一个程序，判断用户输入的是否为回文。

**15-3.** 实现一个程序，计算用户输入中的元音字母个数。

**15-4.** 实现一个支持加法、减法、乘法和除法的计算器程序，能够处理任意两个数字。考虑使用`std::string`的`find`方法和数字转换函数。

**15-5.** 通过以下方式扩展你的计算器程序：允许多种操作或模运算符，并接受浮动小数点数或括号。

**15-6.** 可选：阅读更多关于[本地化]的信息。

**进一步阅读**

+   *ISO 国际标准 ISO/IEC (2017) — 编程语言 C++*（国际标准化组织；瑞士日内瓦；* [`isocpp.org/std/the-standard/`](https://isocpp.org/std/the-standard/) *）

+   *C++编程语言*，第 4 版，作者：Bjarne Stroustrup（Pearson Education，2013）

+   *Boost C++库*，第 2 版，作者：Boris Schäling（XML Press，2014）

+   *C++标准库：教程与参考*，第 2 版，作者：Nicolai M. Josuttis（Addison-Wesley Professional，2012）
