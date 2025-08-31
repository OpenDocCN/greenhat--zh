## 第六章 文本转换与替换

本章将介绍各种单行命令，这些命令可以更改、转换和替换文本，包括 base64 编码和解码、URL 转义和反转义、HTML 转义和反转义、文本大小写转换以及反转行。你还将了解 `y`、`tr`、`uc`、`lc` 和 `reverse` 操作符及字符串转义序列。

## 6.1 对字符串进行 ROT13 编码

```
perl -le '$string = "*bananas*"; $string =~ y/A-Za-z/N-ZA-Mn-za-m/; print $string'
```

这个单行命令使用 `y` 操作符（也叫做 `tr` 操作符）来执行 ROT13 编码。`y` 和 `tr` 操作符执行字符串转译。给定 `y/search/replace/`，`y` 操作符将 `search` 列表中找到的所有字符转译为 `replace` 列表中对应位置的字符。`y` 和 `tr` 操作符通常会被误认为接受正则表达式，但它们并不接受。它们是进行字符转译，并接受 `search` 和 `replace` 部分中的字符列表。

在这个单行命令中，`A-Za-z` 会创建以下字符列表：

```
ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz
```

而 `N-ZA-Mn-za-m` 会创建以下列表：

```
NOPQRSTUVWXYZABCDEFGHIJKLMnopqrstuvwxyzabcdefghijklm
```

注意，在第二个列表中，大写字母和小写字母的偏移量为 13 个字符。现在，`y` 操作符将第一个列表中的每个字符转换为第二个列表中的字符，从而执行 ROT13 操作。（关于 ROT13 的一个有趣事实是，应用两次 ROT13 操作会产生相同的字符串；也就是说，ROT13(ROT13(*string*)) 等于 *string*。）

要对整个文件 *bananas.txt* 进行 ROT13 编码并将结果打印到屏幕上，只需执行以下操作：

```
perl -lpe 'y/A-Za-z/N-ZA-Mn-za-m/' *bananas*.txt
```

你还可以使用 Perl 的 `-i` 参数对文件进行原地替换。例如，要对 *oranges.txt* 进行 ROT13 编码并直接修改文件，请执行以下命令：

```
perl -pi.bak -e 'y/A-Za-z/N-ZA-Mn-za-m/' *oranges*.txt
```

这个单行命令首先创建一个名为 *oranges.txt.bak* 的备份文件，然后用 ROT13 编码的文本替换 *oranges.txt* 的内容。`-i` 命令的 `.bak` 部分创建了备份文件。如果你对结果非常确信，可以省略 `.bak` 部分，但我建议始终使用 `-i.bak`，因为有一天你可能会犯错，弄乱一个重要的文件。（我说的是亲身经历。）

## 6.2 对字符串进行 Base64 编码

```
perl -MMIME::Base64 -e 'print encode_base64("*string*")'
```

这个单行命令使用了 `MIME::Base64` 模块。它导出了 `encode_base64` 函数，该函数接受一个字符串并返回其 base64 编码版本。

要对整个文件进行 base64 编码，请使用以下命令：

```
perl -MMIME::Base64 -0777 -ne 'print encode_base64($_)' *file*
```

在这里，`-0777` 参数与 `-n` 一起使用，导致 Perl 将整个文件加载到 `$_` 变量中。接着，文件被 base64 编码并打印出来。（如果 Perl 没有加载整个文件，它将逐行进行编码，结果会一团糟。）

## 6.3 对字符串进行 Base64 解码

```
perl -MMIME::Base64 -le 'print decode_base64("*base64string*")'
```

`MIME::Base64` 模块还导出了 `decode_base64` 函数，该函数接受一个 base64 编码的字符串并进行解码。

整个文件也可以通过类似的方式进行解码：

```
perl -MMIME::Base64 -0777 -ne 'print decode_base64($_)' *file*
```

## 6.4 对字符串进行 URL 转义

```
perl -MURI::Escape -le 'print uri_escape("*[`example.com`](http://example.com)*")'
```

要使用这行代码，首先需要安装 `URI::Escape` 模块，可以通过在命令行输入 `cpan URI::Escape` 来安装。该模块导出了两个函数：`uri_escape` 和 `uri_unescape`。第一个函数执行 *URL 转义*（有时称为 *URL 编码*），另一个执行 *URL 解转义*（或 *URL 解码*）。现在，要进行 URL 转义，只需调用 `uri_escape($string)`，就完成了！

这行代码的输出是 `http%3A%2F%2Fexample.com`。

## 6.5 对字符串进行 URL 解转义

```
perl -MURI::Escape -le 'print uri_unescape("*http%3A%2F%2Fexample.com*")'
```

这行代码使用了来自`URI::Escape`模块的`uri_unescape`函数来执行 URL 解转义。它解转义之前的代码输出，逆转该操作。

这行代码的输出是 `http://example.com`。

## 6.6 对字符串进行 HTML 编码

```
perl -MHTML::Entities -le 'print encode_entities("*<html>*")'
```

这行代码使用了来自`HTML::Entities`模块的`encode_entities`函数来编码 HTML 实体。例如，你可以将 `<` 和 `>` 转换为 `&lt;` 和 `&gt;`。

## 6.7 对字符串进行 HTML 解码

```
perl -MHTML::Entities -le 'print decode_entities("*&lt;html&gt;*")'
```

这行代码使用了来自`HTML::Entities`模块的`decode_entities`函数。例如，你可以将 `&lt;` 和 `&gt;` 转换回 `<` 和 `>`。

## 6.8 将所有文本转换为大写

```
perl -nle 'print uc'
```

这行代码使用了 `uc` 函数，默认情况下它作用于 `$_` 变量，并返回它包含的文本的大写版本。

你也可以使用 `-p` 命令行选项，它启用了 `$_` 变量的自动打印并对其进行就地修改：

```
perl -ple '$_ = uc'
```

或者，你可以将 `\U` 转义序列应用到字符串插值中：

```
perl -nle 'print "\U$_"'
```

这行代码会将其后的所有内容（或者直到第一次出现 `\E` 为止）转换为大写。

## 6.9 将所有文本转换为小写

```
perl -nle 'print lc'
```

这行代码与前一行类似，`lc` 函数将 `$_` 的内容转换为小写。

你也可以使用转义序列 `\L` 和字符串插值：

```
perl -nle 'print "\L$_"'
```

在这里，`\L` 会将其后的所有内容转换为小写（或者直到第一次出现 `\E` 为止）。

## 6.10 只将每行的第一个字母转换为大写

```
perl -nle 'print ucfirst lc'
```

这行代码首先通过`lc`函数将输入转换为小写，然后使用`ucfirst`只将第一个字符转换为大写。例如，如果你传入一行文本 *foo bar baz*，它会输出 *Foo bar baz*。类似地，如果传入一行 *FOO BAR BAZ*，它首先将整行转换为小写，然后再将第一个字母转为大写，最终输出 *Foo bar baz*。

你可以使用转义码和字符串插值来做同样的事情：

```
perl -nle 'print "\u\L$_"'
```

首先，`\L`将整行转换为小写，然后`\u`将第一个字符转换为大写。

## 6.11 反转字母的大小写

```
perl -ple 'y/A-Za-z/a-zA-Z/'
```

这行代码会改变字母的大小写：大写字母变为小写字母，小写字母变为大写字母。例如，文本 *Cows are COOL* 会变成 *cOWS ARE cool*。转写操作符 `y`（在第 59 页的 6.1 一行解释）创建了一个从大写字母 `A-Z` 到小写字母 `a-z` 的映射，以及从小写字母 `a-z` 到大写字母 `A-Z` 的映射。

## 6.12 将每行转换为标题式大小写

```
perl -ple 's/(\w+)/\u$1/g'
```

这一行代码尝试将字符串转换为标题大小写，意思是每个单词的第一个字母都大写；例如，*This Text Is Written In Title Case*。这行代码通过匹配每个单词`\w+`，并用`\u$1`替换匹配的单词，从而将单词的第一个字母大写。

## 6.13 删除每行开头的空白字符（空格、制表符）

```
perl -ple 's/^[ \t]+//'
```

这一行代码利用替换操作符`s`删除每行开头的所有空白字符。`s/regex/replace/`表示将匹配的`regex`替换为`replace`字符串。在这个例子中，`regex`是`^[ \t]+`，意思是“匹配字符串开头的一个或多个空格或制表符”，而`replace`为空，意味着“将匹配的部分替换为空字符串”。

正则表达式类`[ \t]`也可以用`\s+`替换，以匹配任何空白字符（包括制表符和空格）：

```
perl -ple 's/^\s+//'
```

## 6.14 删除每行末尾的空白字符（空格、制表符）

```
perl -ple 's/[ \t]+$//'
```

这一行代码删除每行末尾的所有空白字符。`s`操作符的正则表达式表示“匹配字符串末尾的一个或多个空格或制表符”。`replace`部分为空，这意味着“删除匹配的空白字符”。

你也可以通过写成以下形式来实现相同的效果：

```
perl -ple 's/\s+$//'
```

在这里，你可以用`\s+`替换`[ \t]+$`，就像在单行代码 6.13 中一样。

## 6.15 删除每行开头和结尾的空白字符（空格、制表符）

```
perl -ple 's/^[ \t]+|[ \t]+$//g'
```

这一行代码结合了单行代码 6.13 和 6.14。它为`s`操作符指定了全局`/g`标志，因为你希望它删除字符串开头*和*结尾的空白字符。如果不指定这个标志，它只会删除开头的空白（如果有空白）或结尾的空白（如果开头没有空白）。

你也可以将`[ \t]+$`替换为`\s+`，得到相同的结果：

```
perl -ple 's/^\s+|\s+$//g'
```

写`\s+`比写`[ \t]+`更简洁。而`s`代表空格，这使得它更容易记住。

## 6.16 将 UNIX 换行符转换为 DOS/Windows 换行符

```
perl -pe 's|\012|\015\012|'
```

这一行代码将 UNIX 换行符`\012`（`LF`）替换为 Windows/DOS 换行符`\015\012`（`CRLF`）每一行。`s/regex/replace/`的一个优点是，它可以使用除正斜杠以外的其他字符作为分隔符。这里，使用竖线分隔`regex`和`replace`，以提高可读性。

换行符通常表示为`\n`，回车符表示为`\r`，但在不同平台上，`\n`和`\r`的含义可能有所不同。然而，UNIX 换行符始终可以表示为`\012`（`LF`），而回车符表示为`\r`（`CR`）。这就是为什么你使用这些数字代码：有时使用灵活的序列更可取，但在这里并不适用。

## 6.17 将 DOS/Windows 换行符转换为 UNIX 换行符

```
perl -pe 's|\015\012|\012|'
```

这一行代码的作用与单行代码 6.16 相反。它将 Windows 换行符（`CRLF`）转换为 UNIX 换行符（`LF`）。

## 6.18 将 UNIX 换行符转换为 Mac 换行符

```
perl -pe 's|\012|\015|'
```

Mac OS 以前使用`\015`（`CR`）作为换行符。这个单行命令将 UNIX 的`\012`（`LF`）转换为 Mac OS 的`\015`（`CR`）。

## 6.19 在每一行中将“foo”替换为“bar”

```
perl -pe 's/foo/bar/'
```

这个单行命令使用`s/regex/replace/`命令，将每一行中第一次出现的`foo`替换为`bar`。

要将所有的`foo`替换为`bar`，请添加全局`/g`标志：

```
perl -pe 's/foo/bar/g'
```

## 6.20 在匹配“baz”的行中将“foo”替换为“bar”

```
perl -pe '/baz/ && s/foo/bar/'
```

这个单行命令大致等价于

```
while (defined($line = <>)) {
  if ($line =~ /baz/) {
    $line =~ s/foo/bar/
  }
}
```

这个扩展的代码将每一行放入变量`$line`中，然后检查该变量中的行是否与`baz`匹配。如果匹配，则将该行中的`foo`替换为`bar`。

你也可以这样写

```
perl -pe 's/foo/bar/ if /baz/'
```

## 6.21 逆序打印段落

```
perl -00 -e 'print reverse <>' *file*
```

这个单行命令使用了在单行命令 2.7（第 14 页）中讨论的`-00`参数，启用段落吸取模式，意味着 Perl 按段落读取文本，而不是按行读取。接着，它使用`<>`运算符让 Perl 从标准输入或指定的文件中读取输入。这里，我指定了`file`作为参数，因此 Perl 将按段落读取`file`（得益于`-00`）。一旦 Perl 读取完文件，它会将所有段落作为一个列表返回，并调用`reverse`来反转段落列表的顺序。最后，`print`打印反转后的段落列表。

## 6.22 逆序打印所有行

```
perl -lne 'print scalar reverse $_'
```

这个单行命令在标量上下文中评估`reverse`运算符。在前面的单行命令中，你看到在列表上下文中评估`reverse`会反转整个列表，也就是元素的顺序。要对像`$_`这样的标量值（包含整行内容）执行相同的操作，你必须在标量上下文中调用`reverse`。否则，它只会反转一个包含单个元素的列表，也就是同样的列表！完成这个操作后，你只需打印反转后的行。

通常，在使用运算符时，你可以省略`$_`变量，Perl 仍然会在`$_`变量上应用该函数。换句话说，你可以将相同的单行命令重写为

```
perl -lne 'print scalar reverse'
```

或者你可以将`-n`替换为`-p`，修改`$_`变量，并将其值设置为反转：

```
perl -lpe '$_ = reverse $_'
```

你也可以这样写

```
perl -lpe '$_ = reverse'
```

这里，`$_`被省略了，因为大多数 Perl 运算符在没有给定参数时，默认使用`$_`。

## 6.23 逆序打印列

```
perl -alne 'print "@{[reverse @F]}"'
```

这个单行命令反转文件中列的顺序。`-a`命令行参数将每一行按空格分割成列，并将它们放入`@F`数组，然后反转并打印出来。这个单行命令类似于第 32 页上的单行命令 4.4；我在那里解释了`@{[ ... ]}`构造。它简单地让你在双引号内运行代码。例如，给定以下输入文件：

```
one two three four
five six seven eight
```

这个单行命令反转了列的顺序，输出如下：

```
four three two one
eight seven six five
```

如果输入中的列是由空格以外的任何字符分隔的，你可以使用`-F`命令行参数来设置不同的分隔符。例如，给定以下输入文件：

```
one:two:three:four
five:six:seven:eight
```

你可以像这样在单行命令中添加`-F:`命令行参数：

```
perl -F: -alne 'print "@{[reverse @F]}"'
```

它会产生如下输出：

```
four three two one
eight seven six five
```

然而请注意，输出中缺少了`:`字符。要恢复它们，你需要稍微修改单行命令，并将`$"`变量设置为`":"`，如下所示：

```
perl -F: -alne '$" = ":"; print "@{[reverse @F]}"'
```

这会产生预期的输出：

```
four:three:two:one
eight:seven:six:five
```

`$"`变量会改变在数组元素间插入的字符，当数组被插入到双引号字符串中时就是这样。
