## 附录 B：在 Windows 上使用 Perl 单行命令

在本附录中，我将向你展示如何在 Windows 上运行 Perl，如何在 Windows 上安装 bash 移植版本，并展示如何通过三种不同的方式使用 Perl 单行命令：通过 Windows 的 bash 移植版本、Windows 命令提示符（*cmd.exe*）以及 PowerShell。

## B.1 Windows 上的 Perl

在 Windows 上运行 Perl 之前，你需要安装适用于 Windows 的 Perl。我最喜欢的 Windows Perl 移植版本是 Strawberry Perl（*[`strawberryperl.com/`](http://strawberryperl.com/)*），这是一个包含你在 Windows 上运行和开发 Perl 应用所需的一切的 Perl 环境。Strawberry Perl 的设计尽可能像 UNIX 系统上的 Perl 环境。它包括 Perl 二进制文件、gcc 编译器及相关构建工具，以及许多外部库。

要安装 Strawberry Perl，下载并运行安装程序，点击几次菜单就可以完成安装。我的安装目录选择是*c:\strawberryperl*。（将任何 UNIX 软件安装到没有空格的目录中是个好主意。）安装完成后，安装程序应该会将安装目录添加到你的路径环境变量中，这样你就可以直接在命令行运行 Perl 了。

不幸的是，与 UNIX 系统的命令行相比，Windows 命令行非常基础。UNIX 系统运行的是一个真正的 shell，具有明确的命令行解析规则，而 Windows 并没有类似的东西。Windows 命令行对于某些符号的处理有奇怪的规则，引用规则不明确，转义规则也很奇怪，这一切都使得运行 Perl 单行命令变得困难。因此，在 Windows 上运行单行命令的首选方法是使用 UNIX shell（如 bash），正如你将在下一节中学到的那样。

## B.2 Windows 上的 Bash

在 Windows 上运行 bash shell 非常简单。我推荐 win-bash（*[`win-bash.sourceforge.net/`](http://win-bash.sourceforge.net/)*），这是一个适用于 Windows 的独立 bash 移植版本，无需特殊环境或额外的 DLL 文件。下载包是一个包含 bash shell（*bash.exe*）和一堆 UNIX 工具（如 awk、cat、cp、diff、find、grep、sed、vi、wc 等大约 100 个工具）的 zip 文件。

要安装 bash 和所有相关工具，只需解压文件即可完成安装。我的安装目录选择是*c:\winbash*，同样是没有空格的目录。从*c:\winbash*运行*bash.exe*以启动 bash shell。

如果在安装了 Strawberry Perl 之后启动*bash.exe*，Perl 应该可以立即使用，因为 Strawberry Perl 的安装程序应该已经将安装目录添加到了路径中。要确认这一点，请运行`perl --version`。它应该输出已安装 Perl 的版本。如果你收到“找不到`perl`”的错误，手动将*C:\strawberryperl\perl\bin*目录添加到`PATH`环境变量中，可以在命令行输入以下内容：

```
PATH=$PATH:C:\\strawberryperl\\perl\\bin
```

Bash 使用 `PATH` 变量来查找可执行文件并运行它们。通过将 Strawberry Perl 的二进制目录添加到 `PATH` 变量中，你告诉 bash 去哪里查找 `perl` 可执行文件。

## B.3 在 Windows Bash 中的 Perl 一行命令

在 Windows 上的 bash 和 UNIX 之间有一些重要的区别。第一个区别与文件路径有关。Win-bash 支持 UNIX 风格和 Windows 风格的路径。

假设你将 win-bash 安装在 *C:\winbash*。当你启动 *bash.exe* 时，它应该会将根目录 */* 映射到当前的 C: 驱动器。要将根目录切换到另一个驱动器，比如 D:，在 bash shell 中输入 `cd d:`。要切换回 C:，在 shell 中输入 `cd c:`。现在，你可以通过 */work/report.txt*、c:/work/report.txt 或 *c:\\work\\report.txt* 访问像 *C:\work\report.txt* 这样的文件。

使用 win-bash 的最大优势是本书中的所有一行命令都应该能够正常工作，因为你正在运行一个真正的 shell，就像在 UNIX 环境中一样！例如，要给 *C:\work\report.txt* 文件的每一行加上行号（第 17 页的单行命令 3.1），你可以运行：

```
perl -pe '$_ = "$. $_"' C:/work/report.txt
```

或者你可以像在 UNIX 中一样引用该文件：

```
perl -pe '$_ = "$. $_"' /work/report.txt
```

或者你也可以使用 Windows 风格的路径：

```
perl -pe '$_ = "$. $_"' C:\\work\\report.txt
```

为了避免使用双反斜杠，你可以用单引号引用文件路径：

```
perl -pe '$_ = "$. $_"' 'C:\work\report.txt'
```

如果文件名中有空格，你必须始终引用它。例如，要操作 *C:\Documents and Settings\Peter\My Documents\report.txt*，在传递给一行命令时，需要引用整个路径：

```
perl -pe '$_ = "$. $_"' 'C:\Documents and Settings\Peter\My Documents\report.txt'
```

或者使用 UNIX 风格的文件路径：

```
perl -pe '$_ = "$. $_"' '/Documents and Settings/Peter/My Documents/report.txt'
```

在这里引用文件名是必要的，因为如果不引用，Perl 会认为你传递的是一堆文件，而不是一个带空格的单一文件。

## B.4 在 Windows 命令提示符中的 Perl 一行命令

如果由于某种原因你无法按推荐的方式使用 win-bash，你可以通过 Windows 命令提示符 (*cmd.exe*) 运行一行命令。如果你在 Windows 命令提示符中运行这些一行命令，你需要稍微修改一下本书中的命令，因为 Windows 解析和处理命令行参数的方式不同。下面是你需要做的。

首先，验证 Perl 是否可以通过命令提示符使用。启动 *cmd.exe* 并在命令行中输入 `perl --version`。如果你在安装 Strawberry Perl 后执行此操作，命令应该会输出 Perl 版本信息，这样就可以正常使用了。否则，你需要通过更新 `PATH` 环境变量来添加 Strawberry Perl 的二进制目录路径：

```
set PATH=%PATH%;C:\strawberryperl\perl\bin
```

和 UNIX 一样，`PATH` 变量告诉命令提示符在哪里查找可执行文件。

### 在 Windows 命令提示符中转换单行命令

现在让我们来看看如何为命令提示符转换一行命令，从一行命令 2.1（第 7 页），它将文件内容双倍间距开始。在 UNIX 中，你只需运行：

```
perl -pe '$\ = "\n"' *file*
```

然而，如果你在 Windows 命令提示符中运行这个一行命令，你必须确保它总是用外部的双引号括起来，并且你已经转义了其中的双引号和特殊字符。做了这些更改后，这个一行命令在 Windows 上应该是这样的：

```
perl -pe "$\ = \"\n\"" *file*
```

这行命令变得很乱，但你可以用一些 Perl 技巧使它看起来稍微整洁一些。首先，用 `qq/.../` 操作符将一行命令中的双引号替换掉，它会将斜杠之间的任何内容加上双引号。在 Perl 中写 `qq/text/` 等价于写 `"text"`。现在你可以这样重写这行命令：

```
perl -pe "$\ = qq/\n/" *file*
```

这要好一些。你还可以改变 `qq` 操作符用于分隔内容的字符。例如，语法 `qq|...|` 会将管道符 `|` 之间的内容加上双引号：

```
perl -pe "$\ = qq|\n|" file
```

你甚至可以使用匹配的圆括号或大括号，如下所示：

```
perl -pe "$\ = qq(\n)" file
```

或者是这样：

```
perl -pe "$\ = qq{\n}" file
```

让我们看看如何将更多的一行命令转换到 Windows。比如将一个 IP 地址转换为整数（第 45 页上的一行命令 4.27）？在 UNIX 中你可以运行：

```
perl -MSocket -le 'print unpack("N", inet_aton("127.0.0.1"))'
```

在 Windows 上，你需要将一行命令外部的引号改为双引号，并且转义一行命令内部的双引号：

```
perl -MSocket -le "print unpack(\"N\", inet_aton(\"127.0.0.1\"))"
```

或者你可以使用 `qq|...|` 操作符，避免在一行命令中转义双引号：

```
perl -MSocket -le "print unpack(qq|N|, inet_aton(qq|127.0.0.1|))"
```

对于不需要插值的内容，如格式字符串 `N` 和 IP 地址 `127.0.0.1`，你也可以使用单引号而不是双引号：

```
perl -MSocket -le "print unpack('N', inet_aton('127.0.0.1'))"
```

另一个技巧是使用 `q/.../` 操作符，它会将斜杠之间的文本单引号化：

```
perl -MSocket -le "print unpack(q/N/, inet_aton(q/127.0.0.1/))"
```

写 `q/N/` 和 `q/127.0.0.1/` 与写 `'N'` 和 `'127.0.0.1'` 是一样的。

让我们将另一个 UNIX 的一行命令转换为 Windows。我已将它扩展为多行以便清晰展示：

```
perl -le '
  $ip="127.0.0.1";
  $ip =~ s/(\d+)\.?/sprintf("%02x", $1)/ge;
  print hex($ip)
'
```

不幸的是，要将其转换为 Windows，你需要将所有行连接起来（这样结果就不太易读了），并应用新的引用规则：

```
perl -le "$ip=\"127.0.0.1\"; $ip =~ s/(\d+)\.?/sprintf(\"%02x\", $1)/ge; print hex($ip)"
```

你也可以通过使用 `qq` 操作符稍微提高可读性：

```
perl -le "$ip=qq|127.0.0.1|; $ip =~ s/(\d+)\.?/sprintf(qq|%02x|, $1)/ge; print hex($ip)"
```

或者通过使用单引号：

```
perl -le "$ip='127.0.0.1'; $ip =~ s/(\d+)\.?/sprintf('%02x', $1)/ge; print hex($ip)"
```

### 符号挑战

你还可能会遇到一行命令中 `^` 符号的问题，因为 Windows 命令提示符将 `^` 作为转义符。为了让 Windows 字面上处理 `^` 符号，你*通常*需要将每个 `^` 替换为两个 `^^`：

让我们看几个简单的例子，看看如何打印 `^` 符号。下面是我的第一次尝试：

```
perl -e "print \"^\""
```

没有输出！`^` 符号消失了。我们再试试输入 `^` 两次：

```
perl -e "print \"^^\""
```

成功了！它打印了 `^` 符号。现在让我们尝试使用单引号：

```
perl -e "print '^'"
```

这也成功了，打印了 `^`，而且我不需要输入两次 `^`。使用 `qq/^/` 也能成功：

```
perl -e "print qq/^/"
```

正如你所见，在 Windows 上运行一行命令可能会有些棘手，因为没有统一的命令行参数解析规则。编写包含 `%`、`&`、`<`、`>` 和 `|` 符号的一行命令时，你可能会遇到类似的问题。如果是这样，可以尝试在这些符号前加上 `^` 转义字符，使 `%` 变成 `^%`，`&` 变成 `^&`，`<` 变成 `^<`，`>` 变成 `^>`，`|` 变成 `^|`。或者，可以尝试将它们包裹在 `qq` 操作符中，正如我之前讨论的那样。（更好的方法是安装 win-bash 并通过它来运行一行命令，以避免所有这些问题。）

### Windows 文件路径

使用 Windows 命令提示符时，你可以通过多种方式将文件名传递给单行命令。例如，要访问文件 *C:\work\wrong-spacing.txt*，你可以输入：

```
perl -pe "$\ = qq{\n}" C:\work\wrong-spacing.txt
```

或者你也可以反转斜杠：

```
perl -pe "$\ = qq{\n}" C:/work/wrong-spacing.txt
```

如果文件名包含空格，你必须对路径进行引号处理：

```
perl -pe "$\ = qq{\n}" "C:\Documents and Settings\wrong-spacing.txt"
```

更多 Windows Perl 使用技巧，请参见 Win32 Perl 文档：[`perldoc.perl.org/perlwin32.html`](http://perldoc.perl.org/perlwin32.html)。

## B.5 PowerShell 中的 Perl 单行命令

在 PowerShell 中运行单行命令与在命令提示符 (*cmd.exe*) 中运行略有不同。主要区别在于 PowerShell 是一种现代的 Shell 实现，其解析规则与命令提示符不同。在本节中，我将展示如何在 PowerShell 中运行 Perl 单行命令。

首先，你需要验证 Perl 是否在 PowerShell 环境中工作。你可以在 PowerShell 中运行 `perl --version`。如果命令输出了 Perl 的版本信息，则表示 Perl 可用，你应该能够运行单行命令。否则，更新 `Path` 环境变量，并通过以下命令将 Strawberry Perl 的二进制目录添加到其中：

```
$env:Path += ";C:\strawberryperl\perl\bin"
```

`Path` 变量告诉 PowerShell 去哪里查找可执行文件，因此当你运行 `perl` 时，它会搜索所有的目录（通过 `;` 字符分隔），找到 *perl.exe*。

### 在 PowerShell 中转换单行命令

参考单行命令 2.1（第 7 页），它将文件进行双倍空格处理。在 UNIX 中，单行命令看起来是这样的：

```
perl -pe '$\ = "\n"' *file*
```

要使这个单行命令在 PowerShell 中运行，你需要改动三个地方：

+   通过在 `$` 符号前添加 `` ` ``（反引号）字符来转义 PowerShell 中用于变量的 `$` 符号：`` `$ ``。

+   与 *cmd.exe* 命令提示符一样，请确保单行命令的外部使用双引号。

+   使用 `qq/.../` 运算符来处理单行命令中的双引号，如第 108 页“在 Windows 命令提示符中转换单行命令”一节所述。然而，你不能像在命令提示符中那样使用反斜杠转义双引号；你必须使用 `qq/.../` 运算符。

当你将所有这些内容组合起来时，这个单行命令在 PowerShell 中的版本将变为：

```
perl -pe "`$\ = qq/\n/" file
```

要指定文件的完整路径，请使用 Windows 风格的路径。例如，要引用位于 *C:\work\wrong-spacing.txt* 的文件，可以直接在单行命令后输入该路径：

```
perl -pe "`$\ = qq/\n/" C:\work\wrong-spacing.txt
```

如果文件名或文件路径包含空格，请这样输入，路径周围加上双引号：

```
perl -pe "`$\ = qq/\n/" "C:\Documents and Settings\wrong-spacing.txt"
```

现在来看这个相同单行命令的另一个版本。在 UNIX 中，单行命令看起来是这样的：

```
perl -pe '$_ .= "\n" unless /^$/' *file*
```

但是在 PowerShell 中，你必须将外部的单引号改为双引号，转义 `$` 符号，并将单行命令中的双引号改为 `qq/.../`：

```
perl -pe "`$_ .= qq/\n/ unless /^`$/" *file*
```

现在让我们看看用于给文件中非空行编号的单行命令（第 18 页的单行命令 3.2）：

```
perl -pe '$_ = ++$x." $_" if /./'
```

转换为 PowerShell 时，单行命令看起来像这样：

```
perl -pe "`$_ = ++`$a.qq/ `$_/ if /./"
```

那么检查一个数字是否是质数的艺术性单行命令（第 29 页的单行命令 4.1）怎么样？

```
perl -lne '(1x$_) !~ /¹?$|^(11+?)\1+$/ && print "$_ is prime"'
```

在 PowerShell 中，单行命令看起来是这样的：

```
perl -lne "(1x`$_) !~ /¹?`$|^(11+?)\1+`$/ && print qq/`$_ is prime/"
```

记得第 46 页提到的将 IP 转换为整数的一行命令吗？这是它在 UNIX 中的写法：

```
perl -le '
  $ip="127.0.0.1";
  $ip =~ s/(\d+)\.?/sprintf("%02x", $1)/ge;
  print hex($ip)
'
```

这是 PowerShell 中相同的一行命令：

```
perl -le "
  `$ip=qq|127.0.0.1|;
  `$ip =~ s/(\d+)\.?/sprintf(qq|%02x|, `$1)/ge;
  print hex(`$ip)
"
```

### PowerShell 3.0+ 中的一行命令

如果你运行的是 PowerShell 3.0 或更高版本，你可以使用 `--%` 转义序列来防止 PowerShell 进行额外的解析。

要查看你正在运行的 PowerShell 版本，在命令行输入 `$PSVersionTable.PSVersion`。它应该输出如下表格：

```
PS C:\Users\Administrator> $PSVersionTable.PSVersion
Major  Minor  Build  Revision
-----  -----  -----  --------
3      0      -1     -1
```

该表格显示你正在运行 PowerShell 3.0 版本，支持 `--%` 转义序列。（旧版本的 PowerShell 不支持此序列，这种情况下你必须使用我之前描述的技巧。）

使用 `--%` 转义序列时，你不需要转义 `$` 符号。它还允许你在一行命令中使用反斜杠转义双引号。例如，以下是使用 `--%` 转义序列的双倍行间距命令：

```
perl --% -pe "$\ = \"\n\""
```

你还可以使用 `qq/.../` 运算符来避免在一行命令中转义双引号：

```
perl --% -pe "$\ = qq/\n/"
```

这是你在 PowerShell 3.0 或更高版本中编写相同一行命令的另一种方式：

```
perl --% -pe "$_ .= \"\n\" unless /^$/" *file*
```

这是给行编号的一行命令的写法：

```
perl --% -pe "$_ = ++$a.qq/ $_/ if /./"
```

这是使用正则表达式判断一个数是否为质数的一行命令：

```
perl --% -lne "(1x$_) !~ /¹?$|^(11+?)\1+$/ && print \"$_ is prime\""
```

这是将 IP 转换为整数的一行命令：

```
perl --% -le "
  $ip=\"127.0.0.1\";
  $ip =~ s/(\d+)\.?/sprintf(\"%02x\", $1)/ge;
  print hex($ip)
"
```

如你所见，在 PowerShell 中运行一行命令相当棘手，并且需要一些变通方法。再次推荐你按照第 106 页的“Windows 上的 Bash”部分安装 win-bash，以避免必须实现这些变通方法。
