

## 第九章：9 文件和目录



![](img/opener.jpg)

许多应用程序需要你从文件中读取或写入数据。在本章中，我们将探索如何通过 PHP 脚本与文件进行交互。我们将主要关注简单的 *.txt* 文件，尽管我们也会简要介绍 PHP 如何处理其他常见的文本文件格式。

PHP 提供了许多内置函数来处理文件。有些函数一次性读取或写入文件，而其他更底层的函数则提供更细粒度的控制，允许你打开和关闭文件，并在特定位置选择性地读取或写入内容。并不是所有的 Web 应用程序都需要你处理外部文件，但掌握这些函数仍然很有用，以防有需要。举例来说，在 Web 应用程序之外，你可能会需要重新格式化文件中的数据，或者移动和重命名文件和目录。通过我们将在这里讨论的函数，你可以编写一个 PHP 脚本来自动化这个过程。

### 将文件读取到字符串中

如果你知道文件存在并且希望将其所有内容作为单个字符串读取到脚本中，你可以通过一个语句实现，只需调用内置的 file_get_contents() 函数。为了说明这一点，让我们首先创建一个待读取的文件。列表 9-1 显示了一个由 Jorge Suarez 编写的编程俳句文件（可以在 *[`selavo.lv/wiki/index.php/Programming_haiku`](http://selavo.lv/wiki/index.php/Programming_haiku)* 找到）。创建一个名为 *data.txt* 的新文件，包含以下内容。

```
what is with this code?
oh my, looks like I wrote it
what was I thinking?
```

列表 9-1：包含编程诗歌的文本文件 data.txt

这个文件包含三行文本。换行符表示前两行以一个不可见的换行符结尾。

现在我们有了一个文件可以操作，我们可以编写一个脚本来读取并打印它的内容。在与 *data.txt* 相同目录下创建一个 *main.php* 文件，并输入 列表 9-2 中的代码。

```
<?php
$file = __DIR__ . '/data.txt';

$text = file_get_contents($file);
print $text;
```

列表 9-2：一个 main.php 脚本，用于读取并打印文件内容

首先，我们声明一个 `$file` 变量，包含文本文件的路径和文件名。由于文本文件和主脚本在同一目录下，我们通过将 __DIR__ 魔术常量（主脚本所在位置的路径）与正斜杠和 *data.txt* 文件名连接起来，构建这个文件位置字符串。然后，我们使用 file_get_contents() 函数将文件内容读取到 `$text` 变量中。最后，我们打印出包含文件内容的字符串。

在终端运行主脚本，你应该看到诗句分布在三行上，正如在示例 9-1 中所显示的那样。这是因为文件中的不可见新行字符被传递到了$text 字符串中，就像可见字符一样。我们可以通过几种方式证明这些不可见字符的存在：检查文本文件的大小，或者通过将字符串中的新行替换为可见字符来验证从文件中读取的内容。为了更清楚地看到新行字符是如何成为文本文件的一部分的，让我们将*data.txt*的内容替换为示例 9-3 中的内容。

```
a
b
```

示例 9-3：一个简化版的 data.txt 文件

现在，文件只包含两个字符，每个字符位于单独的行上，我们可以更轻松地检查文件内容。更新*main.php*以匹配示例 9-4。

```
<?php
$file = __DIR__ . '/data.txt';

$text = file_get_contents($file);

$numBytes = filesize($file);
$newlinesChanged = str_replace("\n", 'N', $text);

print "numBytes = $numBytes\n";
print $newlinesChanged;
```

示例 9-4：一个更新后的 main.php 脚本，用于证明新行字符的存在

和之前一样，我们首先将文件内容读取到$text 变量中。然后，我们使用内建的 filesize()函数读取文件的大小，它返回文件的字节数。在一个只有基本 ASCII 字符的文本文件中，每个字符（包括不可见字符）占用 1 个字节，所以我们应该预期结果是 3。接下来，我们生成另一个字符串，将$text 中的每个新行字符（"\n"）替换为大写字母 N，并将结果存储在$newLinesChanged 变量中。最后，我们打印文件大小和更新后的字符串。以下是运行此脚本后在终端中输出的内容：

```
numBytes = 3
aNb
```

第一行确认文件只包含三个字符（字节）的数据：字母 a、一个新行字符和字母 b。第二行是代表文件内容的字符串，其中的新行字符被替换为可见字符：aNb 再次确认文件只包含三个字符，其中两个字母之间有一个新行字符。

确认新行的存在并不是一项简单的任务：在本章后面，我们将探索逐行处理文件内容的函数。这些函数依赖于不可见的新行字符来判断一行的结束和下一行的开始。

> 注意

*file_get_contents()* *函数也可以从网上读取文件，而不仅仅是从本地机器读取，只需传递文件位置的完整 URL。例如，尝试将 URL* [`filesamples.com/samples/document/txt/sample1.txt`](https://filesamples.com/samples/document/txt/sample1.txt) *存储在* $file *变量中，然后像示例 9-2 中那样调用* file_get_contents($file) *。你应该会得到一串无意义的拉丁文文本。*

#### 确认文件是否存在

之前的示例假设存在一个名为*data.txt*的文件。然而，在实际操作中，最好在尝试读取文件内容之前先测试文件是否存在。否则，如果你尝试打开或读取一个找不到的文件，你将收到类似以下的运行时警告：

```
PHP Warning:  file_get_contents(/Users/matt/nofile.txt): Failed to open
stream: No such file or directory in /Users/matt/main.php on line 4
```

执行将在警告之后继续，如果脚本尝试操作不存在的文件内容，这可能会导致进一步的警告和错误。为了使你的代码更健壮并能应对缺失的文件，你可以使用内置的 file_exists()函数。它返回一个布尔值，确认所提供的文件是否存在。让我们通过更新*main.php*并参考示例 9-5 的内容来试一试。

```
<?php
$file = __DIR__ . '/data.txt';
$file2 = __DIR__ . '/data2.txt';

$text = "file not found: $file";
$text2 = "file not found: $file2";

if (file_exists($file)) {
 $text = file_get_contents($file);
}

if (file_exists($file2)) {
    $text2 = file_get_contents($file2);
}

print $text . "\n";
print $text2 . "\n";
```

示例 9-5：更新后的 main.php 脚本，在读取文件之前确认文件的存在

在这里，我们添加了$file2，它是一个保存不存在的文件路径*data2.txt*的第二个变量。在尝试读取任何内容之前，我们将默认的文件未找到消息分配给$text 和$text2 变量。这样，即使我们未能读取文件内容，这些变量仍然会保存一些内容。接下来，我们使用 file_exists()函数，在两个连续的 if 语句中确保仅在找到相应文件时才尝试读取*data.txt*和*data2.txt*的内容。然后我们打印$text 和$text2 的内容，每个后面跟一个换行符。结果如下：

```
a
b
file not found: /Users/matt/data2.txt
```

由于*data.txt*文件存在，它的内容已被读取到$text 中（替代了默认的文件未找到消息），并被打印出来。与此同时，由于*data2.txt*文件不存在，打印$text2 会显示一条指示文件无法找到的消息。#### “触摸”文件

Linux 和 macOS 具有 touch 文件终端命令，它会将指定文件的最后访问或修改时间戳更新为当前日期时间，或者如果文件不存在，则创建一个空文件。PHP 提供了几乎相同的 touch()函数，它提供了另一种在访问文件之前确保文件存在的方法。如果你不介意文件内容为空，你可以将示例 9-5 中的默认文件未找到消息和 if 语句替换为简单的 touch()语句，如示例 9-6 所示。

```
<?php
$file = __DIR__ . '/data.txt';
$file2 = __DIR__ . '/data2.txt';

touch($file);
touch($file2);

$text1 = file_get_contents($file);
$text2 = file_get_contents($file2);

print $text1 . "\n";
print $text2 . "\n";
```

示例 9-6：更新后的 main.php 脚本，在读取文件之前“触摸”文件

现在我们在使用 file_read_contents()读取文件之前，先将每个文件名传递给 touch()。这让我们可以安全地读取文件，无需使用 if 语句和 file_exists()，因为我们知道如果文件不存在，touch()会创建文件（尽管是空文件）。

#### 确保目录存在

到目前为止，我们一直在处理与执行脚本位于同一目录中的文件，但文件也可能位于不同的目录中。在这种情况下，确认目录是否存在（并且如果不存在则可能需要创建它）非常重要，因为就像缺失的文件一样，不存在的目录会触发运行时警告。PHP 有两个内置函数来处理这种情况：is_dir()返回一个布尔值，确认指定的目录路径是否可以找到，mkdir()则尝试在指定路径创建目录。

> 注意

*mkdir() 函数如果尝试创建的目录已存在，或者基于当前权限设置无法创建该目录，会抛出运行时警告。有关权限的更多信息，请参见第 163 页的《目录和文件权限》一章。*

要尝试这些函数，请按照示例 9-7 中所示更新 *main.php* 的内容。

```
<?php
$dir = __DIR__ . '/var';
$file = $dir . '/data.txt';

if (!is_dir($dir)) {
    mkdir($dir);
}

touch($file);

$text = file_get_contents($file);
print $text;
```

示例 9-7：一个更新后的 main.php 脚本，在目录不存在时创建该目录

我们将目标路径和文件名拆分为两个变量：$dir 存储着文件所在目录的路径，$file 存储着路径加文件名。我们将 $dir 设置为执行脚本所在目录（__DIR__）下的*/var* 子目录；该子目录不存在。if (!is_dir($dir)) 语句检查 $dir 是否*不是*有效的目录路径，如果不是有效路径，则调用 mkdir() 创建该目录。现在我们可以安全地调用 touch() 创建文件，因为我们已经确认目录存在，然后读取文件，因为 touch() 会在文件不存在时创建该文件。

mkdir() 的默认选项是非递归的：如果目标目录的父目录不存在，它将无法创建该目录。然而，该函数有一个可选的递归参数；如果设置为 true，函数将同时创建任何缺失的父目录。示例 9-8 中展示了一个示例。

```
<?php
$dir = __DIR__ . '/sub/subsub';
$file = $dir . '/data.txt';

if (!is_dir($dir)) {
    mkdir($dir, recursive: true);
}

touch($file);

$text = file_get_contents($file);
print $text;
```

示例 9-8：更新后的 main.php 脚本，如果目录缺失则递归创建目录

现在，目录路径包含一个位于当前执行脚本目录下的*/sub* 目录中的*/subsub* 子目录。在 if 语句内，我们调用 mkdir()，并将递归参数设置为 true。这确保了该函数不仅会创建 */subsub* 目录，还会在必要时创建其父目录 */sub*。我们必须将递归设置为命名参数，因为 mkdir() 还接受另一个可选参数来设置新目录的权限，而该参数在函数签名中排在递归参数之前。

### 将字符串写入文本文件

就像你可以使用 file_get_contents() 将文件内容读取到字符串中一样，你也可以使用互逆的 file_put_contents() 函数将字符串内容写入文本文件。如果目标文件不存在，file_put_contents() 会自动创建该文件，因此你不需要事先测试文件名。更新后的 *main.php* 脚本在示例 9-9 中展示了其用法。

```
<?php
$content = <<<CONTENT
    the cat
    sat
    on the mat!
    CONTENT;

$file = __DIR__ . '/newfile.txt';

file_put_contents($file, $content);
$text = file_get_contents($file);
print $text;
```

示例 9-9：一个 main.php 脚本将数据从字符串写入文件

首先，我们声明一个三行 heredoc 字符串 `$content`，使用 `CONTENT` 作为分隔符。然后，我们将 `$file` 变量设置为当前目录路径加上文件名 *newfile.txt*。接下来，我们调用 `file_put_contents()` 函数，将目标文件和要写入该文件的文本传递给它。这应该会创建一个包含 `$content` heredoc 内容的 *newfile.txt* 文件。为了确认文件已创建并包含文本内容，我们使用 `file_get_contents()` 读取文件中的文本，并将其存储到 `$text` 变量中，然后打印出来。以下是结果：

```
the cat
sat
on the mat!
```

输出与原始 heredoc 字符串一致，表明我们成功地将字符串写入 *newfile.txt* 并再次读取出来。

如果你试图写入的文件已经存在，`file_put_contents()` 的默认行为是完全替换（覆盖）该文件的内容。为了避免这种情况，可以在调用该函数时使用 FILE_APPEND 选项。这将把新文本添加到文件现有内容的末尾。列表 9-10 展示了一个示例，更新自列表 9-9。

```
<?php
$newContent = <<<CONTENT
    the rat
    spat
    on the cat!
    CONTENT;

$file = __DIR__ . '/newfile.txt';

file_put_contents($file, $newContent, FILE_APPEND);
$text = file_get_contents($file);
print $text;
```

列表 9-10：一个 main.php 脚本，将文本追加到文件末尾

这次我们创建一个不同的 heredoc 字符串，并通过将 FILE_APPEND 作为第三个参数调用 `file_put_contents()` 将其添加到 *newfile.txt* 文件中。这应该会将字符串追加到文件当前内容之后，输出确认了这一点：

```
the cat
sat
on the mat!
the rat
spat
on the cat!
```

尝试再次运行列表 9-10 中的代码，不使用 FILE_APPEND 选项。你会发现只有 `$newContent` 中的文本出现在输出中，因为文件中已有的文本被覆盖了。

### 管理文件和目录

除了读取和写入文件外，PHP 还提供了帮助管理现有文件和目录的函数。例如，你可以使用 `unlink()` 函数删除文件，或者使用 `rmdir()` 删除整个目录。如果成功，两个函数都会返回 true，否则返回 false。与读取文件一样，在尝试删除文件之前，测试文件或目录是否存在非常重要。否则，如果你对不存在的文件或目录调用 `unlink()` 或 `rmdir()`，你会收到警告（但执行会继续）。列表 9-11 展示了这些函数的实际应用。

```
<?php
$dir = __DIR__ . '/var';
$file = $dir . '/data.txt';

if (!is_dir($dir)) {
    mkdir($dir);
}

touch($file);

var_dump(is_dir($dir));
var_dump(file_exists($file));

unlink($file);
rmdir($dir);

var_dump(file_exists($file));
var_dump(is_dir($dir));
```

列表 9-11：一个 main.php 脚本，用于创建并删除目录和文件

和之前的一些示例一样，我们在两个变量 $dir 和 $file 中声明目标目录和文件名。然后，如果目录尚未存在，我们会创建该目录，并通过 touch() 创建文件。此时，我们应该确保在*/var* 目录中存在 *data.txt* 文件；我们通过调用 is_dir() 和 file_exists() 并使用 var_dump() 来确认这一点。接下来，我们使用 unlink($file) 和 rmdir($dir) 删除文件及其目录。最后，我们再次调用 var_dump()，以确保在脚本执行完毕后，目录和文件都不存在。如果你运行此脚本，你应该看到 true, true, false, false 的输出，确认目录和文件曾经存在并已成功删除。

另一个有用的文件管理函数是 rename()，它用于更改文件或目录的名称。例如，你可以使用以下语句将*oldfile.txt*重命名为*newfile.txt*：

```
rename('oldfile.txt', 'newfile.txt');
```

使用此函数时需要小心，首先要测试旧的文件或目录是否存在。还需要特别注意新文件或目录。如果你正在重命名一个文件，而另一个同名的文件已经存在，那么该文件将被覆盖而不会显示错误或警告，如果你需要被覆盖文件的内容，这可能会造成问题。如果你正在重命名一个目录，而新目录已经存在，则会生成一个警告，这也不是理想的情况，因为最好避免出现警告。如果你正在将文件重命名到另一个目录中，你还应该确保新目录存在，并且如果需要的话，该目录是可写的（这是 Windows 系统的要求）。有关此函数的更多信息，请参见 *[`www.php.net/manual/en/function.rename.php`](https://www.php.net/manual/en/function.rename.php)*。

### 将文件读取到数组中

PHP 内置的 file() 函数将文件的内容读取到一个数组中，而不是单一的字符串，每一行对应数组中的一个元素。当你想对每一行执行某个操作（例如，像以下示例一样显示行的内容及其行号），或者当每一行代表需要处理的数据集中的一个项时，这个功能非常有用，比如逗号分隔值（CSV）文件中的数据。*Listing 9-12* 展示了一个主脚本，演示了 file() 函数的用法。

```
<?php
$file = __DIR__ . '/data.txt';

$lines = file($file);

foreach ($lines as $key => $line) {
    print "[$key]$line";
}
```

Listing 9-12: 一个用于循环遍历并打印文本文件每一行的 main.php 脚本

我们将文件信息（保存在$file 变量中）传递给 file() 函数，后者会将*data.txt*的内容逐行读取到一个名为$lines 的数组中。然后，我们使用 foreach 循环逐个打印数组的每个元素（文件中的一行），并显示其数字键。如果*data.txt*包含*Listing 9-1*中的三行俳句，则输出应该如下所示：

```
[0]what is with this code?
[1]oh my, looks like I wrote it
[2]what was I thinking?
```

你可以将可选的标志作为第二个参数传递给 file() 函数，例如，排除每行末尾的换行符（FILE_IGNORE_NEW_LINES）或完全忽略文件中的空行（FILE_SKIP_EMPTY_LINES）。

### 使用低级文件函数

file_get_contents() 和 file_put_contents() 函数会为你处理所有与文件相关的步骤，比如打开文件、访问文件内容以及再次关闭文件。在大多数情况下，这些函数已经足够满足需求。然而，有时你可能需要更低级地处理文件，可能是逐行，甚至逐字符地处理。在这种情况下，你可能需要通过一系列单独的低级函数调用来显式地管理文件访问的各个步骤。

PHP 的低级文件函数要求你使用 *文件系统指针*（或简称 *文件指针*），这是文件数据位置的引用。在内部，PHP 将文件视为 *字节流*（一个可以线性读取和写入的资源对象），文件指针提供对该字节流的访问。你可以通过调用 fopen() 并传入你想访问的文件路径来获得文件指针。你还需要传入一个字符串，指定 *如何* 与文件进行交互；例如，文件可以只为读取、只为写入、同时为读写等模式打开。表 9-1 显示了指定一些常见 fopen() 模式的字符串。

表 9-1：常见的 fopen() 模式

| 模式字符串 | 描述 | 文件指针位置 | 如果文件不存在的结果 |
| --- | --- | --- | --- |
| 'r' | 仅读 | 文件开头 | 警告 |
| 'r+' | 读写（覆盖） | 文件开头 | 警告 |
| 'w' | 仅写（覆盖） | 文件开头（并通过移除现有内容来截断文件） | 尝试创建文件 |
| 'a' | 仅写（追加） | 文件末尾 | 尝试创建文件 |

操作文件的典型步骤如下：

1.   以适当的模式打开文件并获取文件指针。

2.   如果需要，改变文件指针在文件中的位置。

3.   在文件指针的位置读取或写入。

4.   根据需要重复步骤 2 和 3。

5.   关闭文件指针。

示例 9-13 演示了这个过程。这个脚本通过使用低级的 fopen()、fread() 和 fclose() 函数，达到了与 示例 9-2（将文件内容读取为字符串）相同的效果。

```
<?php
$file = __DIR__ . '/data.txt';

$fileHandle = fopen($file, 'r');
$filesizeBytes = filesize($file);
$text = fread($fileHandle, $filesizeBytes);
fclose($fileHandle);

print $text;
```

示例 9-13：使用低级函数读取文件

首先，我们使用 fopen() 打开 *data.txt*，使用字符串 'r' 来指定只读模式。该函数返回一个文件指针，指向文件的开头，我们将其存储在 $fileHandle 变量中。接下来，我们调用 filesize() 查找文件的大小（以字节为单位）。然后我们调用 fread() 函数，将文件指针和文件大小（$filesizeBytes）传递给它，以将整个文件的内容读取到 $text 变量中。如果我们只想读取文件的一部分，可以在 fread() 函数的第二个参数中指定不同的字节数。（如果文件指针位于文件的某个位置而不是开头，我们也需要指定不同的字节数。）最后，我们通过将文件指针传递给 fclose() 函数来关闭文件。关闭文件可以使其被其他系统进程使用，并在脚本执行过程中发生错误时防止文件被损坏。

本示例展示了一些最常见的低级文件操作函数，但 PHP 还有许多其他函数。例如，fgets() 从当前文件指针位置读取一行（直到下一个换行符），fgetc() 从当前文件指针位置读取一个字符。feof() 函数接受一个文件指针，并根据指针是否处于文件末尾返回 true 或 false。这在如下循环中非常有用：

```
while (!feof($fileResource)) {
    // Do something at current file pointer position
}
```

这里我们使用 NOT 运算符（!）来否定 feof() 的结果，因此循环将持续进行，直到指针到达文件末尾。在这种循环中，我们可能会使用 fgets() 从文件中读取一行，使用 fgetc() 读取下一个字符，或者使用 fread() 读取固定数量的字节。然后，循环中的逻辑将处理读取到的数据（如果成功读取），如果在读取过程中到达文件末尾，循环将终止。

一些函数只是用于操作和更改文件指针。例如，rewind() 将文件指针移回到文件的开头，ftell() 返回文件指针的当前位置信息，表示为从文件开头开始的字节数。fseek() 函数将文件指针移动到文件中的指定位置，该位置相对于当前指针位置、文件开头或文件末尾指定。

### 处理多个文件

让我们通过一个更复杂的示例，将本章迄今讨论的内容结合起来，程序化地从多个文件中提取数据，并将其汇总到一个新的摘要文件中。我们将尝试收集三名玩家的姓名和游戏分数，每个玩家的数据存储在单独的文件中（*joe.txt*、*matt.txt* 和 *sinead.txt*），对数据进行重新格式化，然后写入名为 *total.txt* 的输出文件中。清单 9-14 到 9-16 显示了我们想要处理的三个原始数据文件。

```
Joe
O'Brien

55
```

清单 9-14：joe.txt

```
Matthew

Smith

99
```

清单 9-15：matt.txt

```
 Sinead
Murphy

101
```

清单 9-16：sinead.txt

请注意，每个数据文件的内容有点杂乱，存在随机位置的空行：清单 9-15 以一个空行结尾，清单 9-16 以两个空行开始和结束。尽管如此，每个数据文件的内容顺序是相同的：第一行包含玩家的名字，第二行包含他们的姓氏，第三行包含他们的整数分数。

在输出文件中，我们希望将每个玩家的所有数据合并到一行中，并显示所有三名玩家分数的总和。清单 9-17 展示了最终生成的*total.txt*文件应该如何显示。

```
Player = Joe O'Brien / Score = 55
Player = Matthew Smith / Score = 99
Player = Sinead Murphy / Score = 101
total of all scores = 255
```

清单 9-17：我们想要创建的合并后的 total.txt 文件

为了实现最终结果，我们需要分别处理每个数据文件的不同部分，因此不能仅仅使用 file_get_contents()将整个文件加载为一个字符串。更好的做法是使用 file()函数将每个文件读取为一个包含单独行的数组。

在处理多个文件时，PHP 名为 glob()的函数是一个强大的工具。它返回一个匹配给定模式的文件和目录路径数组。这对于识别并循环遍历指定位置的所有数据文件特别有用。例如，以下语句提供了一个包含*/data*子文件夹中所有*.txt*文件路径的数组，相对于执行脚本所在的位置：

```
$files = glob(__DIR__ . '/data/*.txt')
```

*是一个通配符，代表任意数量的字符，因此'/data/*.txt'将匹配给定文件夹中任何以*.txt*扩展名结尾的文件名。这正是我们在这个示例中收集玩家数据文件所需要的。

启动一个新项目，并创建一个包含前面在清单 9-14 至 9-16 中展示的文本文件*joe.txt*、*matt.txt*和*sinead.txt*的*/data*子文件夹。然后，在主项目文件夹中，创建一个名为*main.php*的脚本，并包含清单 9-18 中的内容。

```
<?php
$dir = __DIR__ . '/data/';
$fileNamePattern = '*.txt';
$files = glob($dir . $fileNamePattern); ❶

$outputFile = __DIR__ . '/total.txt';
touch($outputFile);
unlink($outputFile);

$total = 0;
foreach ($files as $file) {❷
    $lines = file($file, FILE_IGNORE_NEW_LINES | FILE_SKIP_EMPTY_LINES);
    $firstName = $lines[0];
    $lastName = $lines[1];
    $scoreString = $lines[2];
    $score = intval($scoreString);

    $outputFileHandle = fopen($outputFile, 'a');
    fwrite($outputFileHandle, "Player = $firstName $lastName / Score = $score\n"); ❸
    fclose($outputFileHandle);

    $total += $score;
}

$outputFileHandle = fopen($outputFile, 'a');
fwrite($outputFileHandle, "total of all scores = $total");
fclose($outputFileHandle);

print file_get_contents($outputFile); ❹
```

清单 9-18：处理多个文件的脚本

我们首先将执行脚本所在位置的*/data*子文件夹路径赋值给$dir 变量，并将文件名模式字符串'*.txt'赋值给$fileNamePattern，使用*通配符表示任何*.txt*文件。接着，我们调用 glob()函数，获取$dir 中与$fileNamePattern 匹配的所有文件数组，并将结果存储在$files 变量中❶。由于 glob()函数的帮助，我们知道$files 数组中的所有文件都存在，因此可以避免在尝试读取文件之前检查它们是否存在的麻烦。

接下来，我们将*total.txt*的路径赋值给$outputFile 变量。这个文件可能已经存在，也可能不存在，但我们希望每次运行脚本时都能生成一个新的输出文件。因此，我们使用 touch()函数来创建文件（如果它尚不存在），然后使用 unlink()函数删除该文件。现在，我们可以确保在将数据合并到*total.txt*时，文件是空的。

在将$total 变量初始化为 0 之后，我们使用 foreach 循环❷遍历$files 数组中的文件路径，将每个路径存储到临时变量$file 中。对于每个文件，我们使用 file()函数将其内容读取到一个名为$lines 的数组中。使用 FILE_IGNORE_NEW_LINES 和 FILE_SKIP_EMPTY_LINES 标志调用该函数，确保忽略行尾字符并排除空行。这意味着根据我们对每个数据文件的了解，$lines 应该是一个包含三个元素的数组：第一个元素是玩家的名字，第二个元素是他们的姓氏，第三个元素是他们的分数（以字符串形式表示）。我们从数组中读取这些值到单独的$firstName、$lastName 和$scoreString 变量，并使用内置的 intval()函数将分数从字符串转换为整数。

仍然在 foreach 循环内，我们调用 fopen()来获取输出文件（*total.txt*）的文件指针，采用写附加模式（通过模式字符串'a'指定），这意味着指针将定位到文件末尾。第一次通过循环时，*total.txt*文件不会存在，因此 fopen()会创建该文件。然后，我们使用 fwrite()将一个字符串附加到输出文件中，总结玩家的姓名和分数，并以换行符（\n）结束❸。我们通过 fclose()关闭输出文件，并将当前玩家的分数添加到$total 变量中。

最后，在 foreach 循环完成后，我们再次以写附加模式访问输出文件，并附加一个包含$total 值的最终字符串。然后，为了确保一切正常工作，我们调用 file_get_contents()函数读取输出文件到一个字符串中并打印结果❹。注意，我们直接从打印语句中调用该函数，而不是先将字符串存储到一个变量中。

如果运行*main.php*脚本，你应该得到之前在列表 9-17 中显示的*total.txt*文件。事实上，你可以随意多次运行该脚本，结果将始终相同，因为任何现有的*total.txt*文件都会在 touch()和 unlink()函数的组合操作下被删除。

严格来说，我们的*main.php*脚本并不是实现所需逻辑的最高效方式。我们不需要在每次执行 foreach 循环时都打开和关闭输出文件；我们可以在循环之前只打开一次文件，然后在附加总分后再关闭它。然而，每次通过循环时打开文件可以说明写附加模式的价值，这种模式将文件指针放置在文件的末尾。这样，任何新写入文件的内容都会被添加到现有内容之后。

### JSON 和其他文件类型

PHP 不仅能处理 *.txt* 文件。例如，它还可以处理 JavaScript 对象表示法（JSON）以及其他基于文本的数据格式。对于 JSON 数据，内置的 json_encode() 函数可以将 PHP 数组转换为 JSON 字符串，而 json_decode() 函数则执行相反的操作。这种类型的转换特别顺畅，因为 JSON 数据与 PHP 数组一样，都是围绕键/值对构建的。示例 9-19 展示了这些函数的实际应用。

```
<?php
$filePath = __DIR__ . '/data.json';

$data = [
    'name' => 'matt',
    'office' => 'E-042',
    'phone' => '086-111-2323',
];

$jsonString = json_encode($data);

file_put_contents($filePath, $jsonString);

$jsonStringFromFile = file_get_contents($filePath);
print $jsonStringFromFile;

$jsonArrayFromFile = json_decode($jsonStringFromFile, true);
print "\n";
var_dump($jsonArrayFromFile);
```

示例 9-19：一个将数组转换为 JSON 以及将 JSON 转换回数组的脚本

我们将 *data.json* 的路径存储在 $filePath 变量中。然后我们声明一个 $data 数组，将 'matt'、'E-042' 和 '086-111-2323' 分别映射到键 'name'、'office' 和 'phone'。接下来，我们使用 json_encode() 函数将数组转换为 JSON 格式的字符串，并将结果存储在 $jsonString 变量中。然后，我们使用 file_put_contents() 将 JSON 字符串写入 *data.json* 文件，就像我们写入 *.txt* 文件一样。

脚本的其余部分执行相同的反向操作。我们使用 file_get_contents() 从文件中读取 JSON 数据到 $jsonStringFromFile 变量中，并将其打印出来。该变量包含一个 JSON 字符串，但我们使用 json_decode() 将字符串转换为 PHP 数组，并通过 var_dump() 展示。我们需要提供 true 作为 json_decode() 函数的第二个参数，否则结果将是一个对象类型，而不是数组类型。以下是运行此脚本时在终端的输出：

```
{"name":"matt","office":"E 042","phone":"086 111 2323"}
array(3) {
  ["name"]=>
  string(4) "matt"
  ["office"]=>
  string(5) "E-042"
  ["phone"]=>
  string(12) "086 111 2323"
}
```

第一行展示了我们写入并从 *data.json* 文件中读取的 JSON 字符串。该字符串由一个 JSON 对象组成，使用大括号括起来，包含三个由逗号分隔的键/值对。键与其对应的值通过冒号分隔。其余部分显示了 $jsonArrayFromFile 的内容，这是通过解码 JSON 数据创建的数组。注意 JSON 对象中的键/值对与 PHP 数组中的键/值对之间的直接对应关系。

对于 YAML（YAML 不是标记语言）文本数据文件，PHP 提供了多个函数。例如，yaml_parse() 和 yaml_emit() 函数类似于 json_decode() 和 json_encode()，用于在 YAML 字符串和 PHP 数组之间转换。PHP 还提供了直接的文件到字符串和字符串到文件的 YAML 函数：yaml_parse_file() 和 yaml_emit_file()。

对于 CSV 文件，PHP 提供了直接的文件到字符串和字符串到文件的函数 fgetcsv() 和 fputcsv()。str_getcsv() 函数接受一个 CSV 格式的字符串并将其转换为数组。然而，该函数存在一些缺陷。例如，它不会转义换行符，因此无法处理来自电子表格（如 Google Sheets 或 Microsoft Excel）的典型 CSV 文件。可能正因为这样 PHP 在处理 CSV 数据时不符合标准，导致它没有一个相应的函数来从数组创建 CSV 编码的字符串。

使用可扩展标记语言（XML）要复杂一些。PHP 用对象表示 XML 数据，因此你需要掌握面向对象编程的基础知识，才能使用像 simplexml_load_file() 这样的函数和 SimpleXMLElement 这样的类。然而，一旦你掌握了这些语言特性，PHP 提供了几种强大的方法来遍历和操作 XML 数据。我们将在 Part V 中讨论面向对象的 PHP。

### 总结

在本章中，我们使用了基本的 PHP 函数，如 file_get_contents() 和 file_put_contents()，用于读取和写入外部文件的数据。我们还讨论了 file() 函数，它将文件的每一行读取为单独的数组元素，以及像 fread() 和 fwrite() 这样的低级函数，它们允许你通过指针遍历文件。我们探讨了如何在与文件交互之前，确保文件或目录存在（或不存在），以及如何使用 glob() 获取所有匹配特定标准的文件的引用。虽然我们大部分时间都在处理 *.txt* 文件，但我们也涉及了一些 PHP 函数，用于处理 JSON、YAML、CSV 和 XML 数据格式。

### 练习

1.   在网上找一首打油诗，或者自己写一首。我找到了一首：

```
A magazine writer named Bing
Could make copy from most anything
But the copy he wrote
of a ten-dollar note
Was so good he now lives in Sing Sing
```

编写一个脚本，声明一个数组；数组的每个元素是打油诗中的一行。然后将这些行写入一个名为*limerick.txt*的文本文件中。

2.   在网上找到一个可以通过 URL 访问的示例 JSON 文件（例如，* [`jsonplaceholder.typicode.com`](https://jsonplaceholder.typicode.com) *）。编写一个脚本，从 URL 读取 JSON 字符串，将其转换为数组，然后使用 var_dump() 显示该数组。

3.   在 *data* 文件夹中为游戏玩家及其最高分添加一个新的数据文件，以供 Listing 9-18 中的脚本处理。运行主脚本，你应该会看到输出文件中多了一行，并且新分数已添加到总分中。
