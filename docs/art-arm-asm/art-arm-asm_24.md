

## 第二十章：D THE BASH SHELL INTERPRETER



![](img/opener.jpg)

*Bourne-again shell*，也称为 *Bourne shell* 或 *bash*，是一个 Unix shell 解释器。Bash 是著名的 Unix sh（shell）程序的升级版，后者是 Unix System 7 中的默认 shell 程序。

Bash 是 Linux 系统中典型的 shell（尽管还有其他 shell，如 zsh 和 csh）。大多数 Linux 和 macOS shell 程序在进行简单命令行操作时大致兼容，但它们在支持复杂 shell 编程方面有所不同。

本文中的所有编程示例通过 bash 命令行运行 GCC 和 Gas 汇编器。因此，你至少需要具备一些基本的 bash 知识，以便理解本书中的基本命令。本附录提供了使用 bash 的说明，包括对其常用命令的描述，但这部分内容同样适用于其他 shell 解释器。

为了避免必须提及特定的操作系统或发行版名称，我将在本附录中使用 *Unix* 这个名称来指代底层系统（在撰写本文时，Unix 是 The Open Group 的注册商标）。

## D.1 运行 Bash

bash shell 解释器是一个类似于其他 Unix 应用程序的应用程序。要使用它，你必须首先执行 bash 应用程序。在基于文本的 Unix 系统中，在你登录系统后，会运行某种 shell 应用程序。你可以设置你的系统，使其自动运行 bash（或任何其他 shell 程序）。

在基于 GUI 的 Linux 系统或 macOS 系统中，通常需要运行终端程序来启动一个 shell 解释器。在这两种情况下，通常在 shell 应用程序运行时会呈现一个 *命令提示符*。根据你是否以普通用户或 root 用户身份登录，提示符通常为 $ 或 #；某些 shell 显示 % 提示符。在打印出命令提示符后，shell 将等待你输入命令。

此时，你正在运行一个 shell 解释器，尽管它可能不是 bash；它可能是标准的 sh shell 或其他 shell（例如，macOS 的 *终端* 应用程序运行 zsh）。尽管大多数 shell 的行为大致相同，但为了确保你正在运行 bash，请在命令提示符后输入以下行（然后按 ENTER）：

```
$ bash
```

这将确保你正在运行 bash 应用程序，这样本附录中的所有注释都将适用。你可以通过在命令行中执行退出命令来终止这个 bash 实例并返回到原始 shell。

## D.2 命令行

前面的示例（bash）是给 shell 解释器的命令实例。*命令* 由在命令提示符后输入的一行文本组成（通常通过键盘输入），并采取以下形式：

```
`commandName optionalCommandLineArguments optionalRedirection`
```

这一整行代码被称为*命令行*，它由三个组件组成：一个命令（通常是一个单词，例如前面的 bash），后跟可选的命令行参数，最后是可选的重定向或管道操作数。

命令是内置 bash 命令的名称，或者是可执行应用程序（或 shell 脚本）的名称。例如，命令可能是你刚刚创建的汇编语言源文件的名称。

### D.2.1 命令行参数

*命令行参数*是由空格或制表符分隔的字符字符串，bash 会将这些字符串传递给应用程序。这些命令行参数的确切语法依赖于应用程序。有些简单的应用程序（包括本书中的汇编语言示例）可能完全忽略任何命令行参数；而其他应用程序可能需要非常特定的参数，如果语法不正确，则会报告错误。

一些应用程序定义了两种类型的命令行参数：选项和参数。历史上，命令行*选项*由一个连字符（-）前缀跟随一个单字符，或者由一个双连字符（--）前缀跟随一系列字符。例如，bash 命令支持该选项。

```
bash --help
```

它显示帮助信息并终止，而不会运行 bash 解释器。

相比之下，一个实际的命令行*参数*通常是一个文件名或其他应用程序将作为输入值使用的单词（或字符串）。考虑以下命令行：

```
bash script
```

在这个例子中，bash 将启动一个第二实例，并从*脚本*文件中读取一组命令（每行一个），然后像键盘输入一样执行这些命令。出现在第 38 页的*构建*脚本文件是一个很好的 shell 脚本示例。

由于 bash 使用空格或制表符来分隔命令行参数，如果你想指定一个包含这些*定界符*（在命令行中分隔项目的字符）的单一参数，就会出现问题。幸运的是，bash 提供了一种语法，允许你在命令行中包含这些定界符：如果你用引号括起命令行参数，bash 会将引号内的所有内容作为一个单一的命令行参数传递给应用程序。

例如，如果一个命令需要一个文件名，而你希望使用的文件名包含空格，你可以像下面这样将该文件名传递给命令：

```
`command` "`filename with spaces`"
```

Bash 不会将引号包含为它传递给命令的参数的一部分。如果你需要将引号字符作为命令行参数的一部分传递，请在引号前加上反斜杠字符（\）。例如，考虑以下命令：

```
`command` "`argument containing` \"`quotes`\""
```

这样将包含“引号”的参数作为一个单一参数传递给命令。

正如你将在本附录后面看到的，你还可以用单引号（撇号字符）括起命令行参数。这两者的区别在于变量扩展。

### D.2.2 重定向和管道参数

Bash 程序（以及类 Unix 操作系统）提供了标准输入设备、标准输出设备和标准错误设备。

*标准输入设备*通常是控制台键盘。如果一个程序从标准输入设备读取数据，程序将会暂停，直到用户从键盘输入一行文本。

*标准输出设备*是控制台显示器。如果一个应用程序将数据写入标准输出设备，系统会将其显示在屏幕上。标准错误设备也默认是控制台显示器，因此写入标准错误设备的数据也会显示在屏幕上。

Bash shell 提供了通过在命令行使用特殊参数来*重定向*输入和输出的能力。I/O 重定向通常允许你指定一个文件名。当重定向标准输入设备时，应用程序将从文件中读取文本行（而不是从键盘）。当重定向标准输出时，应用程序将数据写入文本文件，而不是写入控制台显示器。

要从文件重定向标准输入，请使用如下命令行参数：

```
`command` <`InputFile`
```

其中 InputFile 是一个包含将被应用程序读取的文本的文件名。每当应用程序命令通常会从键盘读取一行文本时，它将从指定的文件中读取该行文本。

要将标准输出重定向到文件，请使用以下命令行语法：

```
`command` >`OutputFile`
```

任何通常写入标准输出设备（显示器）的输出将会写入指定的文件（OutputFile）。该语法会删除任何名为*OutputFile*的现有文件内容，并将其内容替换为命令应用程序的输出。

一种输出重定向的变体是将程序的输出附加到现有文件的末尾，而不是替换其内容。要以这种方式使用输出重定向，请使用以下语法：

```
`command` >>`OutputFile`
```

请注意，重定向标准输出设备不会改变标准错误输出设备。如果你已经将标准输出重定向到一个文件，并且某个应用程序将数据写入标准错误设备，那么该输出仍然会显示在控制台上。你可以使用以下语法来重定向标准错误设备：

```
`command` 2>`ErrorOutput`
```

2>告诉 bash 将输出重定向到文件句柄 2。 在类 Unix 系统中，句柄 0 保留用于标准输入，句柄 1 保留用于标准输出，而句柄 2 保留用于标准错误输出设备。将句柄号放在>符号前面，指定要重定向的输出。

如果你愿意，你也可以使用这种语法来重定向标准输出：

```
`command` 1>`OutputFile`
```

最终形式的输入/输出重定向是*管道*，它将一个应用程序的标准输出连接到第二个应用程序的标准输入。这允许第二个应用程序将第一个应用程序的输出作为输入进行读取。以下是管道重定向的语法：

```
`command1` `OptionalCommand1Arguments` | `command2` `OptionalCommand2Arguments`
```

这告诉 bash 将命令 1 的输出重定向为命令 2 的输入。

## D.3 目录、路径名和文件名

当你运行 bash 时，它会默认指向操作系统文件结构中的当前目录。Unix 称此为 *当前工作目录*。每当你运行 bash（例如，当你第一次登录时），当前工作目录通常是你的主目录（由操作系统决定）。例如，在我的 Debian 系统中，这是 */home/rhyde*（在 macOS 下，它是 */Users/rhyde*）。

当你在命令行中指定的文件名不包含任何斜杠字符时，bash 或应用程序会假设该文件存在于系统提供的可执行路径中。*路径名* 是由一个或多个目录名组成的序列，目录名之间用斜杠分隔，最后是文件名。相对路径名以目录名开始；系统会在当前工作目录中查找该目录。例如，*dir1/dir2/filename* 指定了当前工作目录下的 *dir1* 目录中的 *dir2* 中的文件 (*filename*)。*绝对路径名* 以斜杠开始，后面跟着最外层的根目录。例如，*/home/rhyde/x.txt* 指定了在 */home/rhyde* 目录中的 *x.txt* 文件（这是我在 Debian 下的主目录）。

波浪号特殊字符 (~) 是当前用户主目录的简写。因此，*~/x.txt* 是我在 Debian 系统中指定 */home/rhyde/x.txt* 的另一种方式。这个方案在 macOS 中也适用，因此它是以系统独立的方式指定用户目录路径的一个有用方法。

特殊字符句点 (.) 本身是当前工作目录的简写。双句点序列 (..) 是指当前工作目录的上级目录（父目录）。例如，*../x.txt* 指的是父目录中的名为 *x.txt* 的文件。在基于 Linux 的系统中，要从当前工作目录执行应用程序，必须指定 *./filename*，而不能仅仅使用 *filename*（除非你已经将 *./* 添加到执行路径中）。

一些 Unix 命令允许你在命令行中指定多个文件名。这些命令通常允许使用通配符字符来指定多个名称。Unix 在指定通配符时支持一套丰富的正则表达式，其中你最常使用的是星号 (*)。Bash 将匹配星号所代表的任意数量的字符（零个或多个）。因此，文件名 **.txt* 将匹配以四个字符序列 *.txt* 结尾的任何文件（包括仅为 *.txt* 的文件）。

## D.4 内置和外部 Bash 命令

Bash 支持两种类型的命令：内建命令和外部命令。*内建命令*作为 bash 应用程序的一部分存在；一个位于 bash 源代码中的函数处理给定的内建命令。*外部命令*是指与 bash 分开存在的可执行程序，bash 会加载并执行这些程序（然后在这些程序终止后重新接管控制）。内建命令在您运行 bash 时始终可用，但外部命令可能可用，也可能不可用，这取决于这些命令的可执行代码是否存在。除非另有说明，您可以假设以下小节中出现的命令都是外部命令。

本书中的汇编语言示例程序是外部命令的示例。当您输入类似的命令时：

```
./Listing1-5
```

在命令行中，bash 会在当前工作目录 (./) 中找到 *Listing1-5* 可执行文件并尝试执行该代码。

出于安全原因，bash 不会自动执行当前工作目录中的程序，除非您明确地在可执行文件名前加上 ./ 字符。bash 假设没有显式路径信息的程序名可以在 *执行路径* 中找到。执行路径（见第 D.6.1 节，“定义 Shell 脚本变量和值”，在第 961 页）是一个目录列表，bash 会在这些目录中查找您指定的可执行程序，而无需显式的路径信息。通常，bash 会在 */bin*、*/usr/bin* 和 */sbin* 等地方查找可执行程序。

为了使 bash 能够从当前工作目录执行程序，而无需在可执行文件名前加上 ./ 字符，您可以将 ./ 添加到执行路径中。然而，出于安全原因，不建议这么做。有关更多信息，请参阅第 D.8 节，“更多信息”，在第 968 页。

## D.5 基本 Unix 命令

在本附录中描述所有 Unix 命令是不可能的。这将单独需要一本大书。本节描述了对开发汇编语言程序有用的几个命令，以及它们的一些选项和参数。有关其他 bash 命令的信息，请查看第 D.8 节，“更多信息”，在第 968 页。

### D.5.1 man

如果您知道命令名称，但不确定其命令行参数和选项的语法，可以使用 man 命令来了解它。此命令会显示一个（支持的）命令的手册页面，语法如下：

```
man `CommandName`
```

其中 CommandName 是您想要阅读其手册页面的命令名称。例如，下面的命令会显示 man 命令本身的手册页面：

```
man man
```

您可以使用 man 命令，配合以下小节中列出的命令名称，来了解每个命令的更多信息。

### D.5.2 cd 或 chdir

你可以使用 cd（改变目录）命令设置当前工作目录（chdir 是此命令的别名）。标准语法是

```
cd `DirectoryPath`
```

其中 DirectoryPath 是文件系统中某个目录的相对或绝对路径。如果目录不存在或指定的是文件而不是目录，Unix 会报告错误。

如果你不提供任何参数而指定 cd 命令，它会切换到当前用户的主目录。这等同于在命令行中输入以下命令：

```
cd ~
```

cd 和 chdir 命令是内建在 bash 中的。

### D.5.3 pwd

pwd（打印工作目录）命令打印当前工作目录的路径。Bash 通常会在命令行提示符中显示当前工作目录；如果你的系统是这种配置，可能不需要使用 pwd。这个命令也是一个内建的 bash 命令。

### D.5.4 ls

ls（列出目录）命令将目录列表打印到标准输出。没有选项时，它会显示当前目录的内容。

当打印目录列表到显示器时，ls 默认使用多列格式。如果你将输出重定向到文件，或者通过管道使用，命令会以单列格式打印列表。

如果你提供一个目录路径作为参数，ls 命令会显示指定目录的内容（假设该目录存在）。如果你提供一个文件的路径作为参数，ls 命令会只显示该文件名（同样，假设文件在指定路径下存在）。

默认情况下，ls 命令不会显示以句点开头的文件名。Unix 将这些文件视为*隐藏文件*。如果你想显示这些文件名，可以使用 -a 命令行选项：

```
ls -a
```

默认情况下，ls 命令只列出指定目录中的文件名和目录名。如果你指定了 -l（长格式）选项，ls 命令将显示每个文件的更多信息：

```
$ ls -l
total 3256
-rw-r--r--@ 1 rhyde  staff   168089 Dec 29  20`xx` encoder.pdf
-rw-r--r--@ 1 rhyde  staff  1492096 Dec 27  20`xx` mcp23017.png
```

列表中的第一列指定文件权限。接下来的三列提供链接计数和所有权信息，接着是文件大小和修改日期与时间，最后是文件名。

### D.5.5 file

与 macOS 和 Windows 不同，Unix 不会将特定数据类型与文件关联。你可以使用 Unix 的 file 命令来确定某个文件的类型：

```
file `pathname`
```

file 命令会根据指定路径的文件类型做出最佳猜测并返回结果。

### D.5.6 cat, less, more, 和 tail

要查看文本文件的内容，可以使用 cat（连接）命令完整显示该文件内容。

```
cat `pathname`
```

其中 pathname 是你希望显示的文件的路径名。

`cat`命令的问题在于，它试图一次性将整个文件写入显示器。许多文件的大小超出了屏幕一次性显示的范围，因此`cat`最终只会显示文件的最后几行；此外，非常大的文件可能需要一段时间才能显示其内容。如果你希望能够逐屏翻阅文件，可以使用`more`和`less`命令：

```
more `pathname`
less `pathname`
```

`more`命令现在已经过时，但仍然可以处理包含该命令的旧脚本文件。它一次显示一屏的文本，并允许你按行（按 ENTER 键）或按页（按空格键）滚动文件。`more`的一个大缺点是你只能向前查看文件；当信息滚动出屏幕后，它就丢失了。

`less`命令（其名称来自短语*less is more*）是`more`的升级版本，允许你在一页中向前和向后滚动。大多数人使用`less`命令而不是`more`，因为它有更多的功能（比如能够使用箭头键来持续按行上下滚动）。

如果你只想查看大型文件的最后几行，可以使用`tail`命令：

```
tail `pathname`
```

默认情况下，`tail`会打印文件的最后 10 行。你可以使用`-n xxxx`命令行选项，其中`xxxx`是一个十进制数值，用来指定不同的行数。例如

```
tail -n 20 x.txt
```

显示文件*x.txt.*的最后 20 行

### D.5.7 mv

`mv`（移动）命令的语法如下：

```
mv `SourcePath DestinationPath`
```

SourcePath 是你想要移动或重命名的文件的路径，DestinationPath 是你希望文件移动到的最终目标路径（或你希望用于文件的新名称）。

要在当前目录中重命名文件，`mv`命令的形式为

```
mv `OldName NewName`
```

其中，OldName 是你想要更改的现有文件名，NewName 是你想要为文件重命名的新文件名。这两个都是简单的文件名（没有目录路径成分）。请注意，NewName 必须与 OldName 不同。

要将文件从一个目录移动到另一个目录，SourcePath 或 DestinationPath（或两者）必须包含目录成分。SourcePath 必须在路径的末尾包含文件名成分（即要移动的文件名）。对于 DestinationPath，末尾的文件名是可选的。如果 DestinationPath 是一个目录的名称（而不是文件），`mv`将把源文件移动到目标目录，并使用与原始源文件相同的文件名。如果 DestinationPath 末尾有文件名，那么`mv`会在移动文件时更改文件名。

你可以使用通配符字符与`mv`命令一起使用，但须遵守以下限制：通配符字符只能出现在源路径中，目标路径必须是一个目录，而不能是实际的文件名。

### D.5.8 cp

`cp`命令的语法如下：

```
cp `SourcePath DestinationPath`
```

此命令将指定的 SourcePath 文件复制，并使用 DestinationPath 作为复制文件的名称。如果两个路径名都是简单的文件名（即你正在复制当前目录中的文件），则两个文件名必须不同。

cp 命令接受源操作数中的通配符字符。如果存在通配符字符，则目标必须是目录路径。cp 命令将把所有匹配通配符的文件复制到指定的目录中。

如果源和目标操作数都指定了目录，请使用 -R（递归）命令行选项。这将把源目录中的所有文件复制到目标目录中同名的目录（如果目标目录中尚不存在该目录，则会创建新目录）；它还将递归地将源目录中的任何子目录复制到目标目录中同名的子目录中。

### D.5.9 rm

rm 命令从目录中移除（删除）一个文件，使用以下语法：

```
rm `pathname`
```

pathname 参数必须是指向单个文件的路径，而不是目录。要删除目录及其中的所有文件，请使用以下命令：

```
rm -R `DirectoryPath`
```

这将递归地删除 DirectoryPath 中的所有文件和子目录，然后删除由 DirectoryPath 指定的目录。

要删除目录中的所有文件而不删除目录本身，请使用以下命令：

```
rm -R `DirectoryPath/*`
```

使用 rm 命令时要非常小心通配符字符。根据当前工作目录，以下命令可能会删除存储设备上的所有内容：

```
rm -R *
```

还有一个 rmdir 命令可以用来删除空目录。然而，rm -R directory 命令更容易用于这个目的。

### D.5.10 mkdir

mkdir 命令创建一个新的（空的）目录，使用以下语法：

```
mkdir `DirectoryPath`
```

其中 DirectoryPath 指定一个尚不存在的目录的路径名。如果 DirectoryPath 是一个实际的路径名，则路径中的所有子目录名称必须存在，路径中的最后一个目录名称（在最后的 / 之后）不能存在。如果指定的是简单的目录名（没有路径），bash 将在当前工作目录中创建该目录。

mkdir 命令支持 -p 命令行选项，可以创建路径中所有不存在的目录。

### D.5.11 date

date 命令显示当前日期和时间。你也可以使用此命令来设置 Unix 实时时钟。运行 man date 获取详细信息。

### D.5.12 echo

echo 命令将命令行其余部分的文本（由 bash 扩展）打印到标准输出设备。例如：

```
echo hello, world!
```

将会把 hello, world! 写入标准输出。你将在脚本中或用来显示各种 shell 变量的值时最常使用此命令。

### D.5.13 chmod

尽管 Unix 文件没有具体类型，但目录确实会维护文件是否可以由文件所有者、与文件相关联的组或任何人读取、写入或执行（标准的 Unix 权限）。chmod 命令允许你为特定文件设置（或清除）权限模式位。

chmod 的基本语法是

```
chmod `options pathname`
```

其中 pathname 是你想要更改模式的文件路径，options 参数指定新的权限。

options 参数可以是一个八进制（基数 8）数字（通常为三位数），也可以是一个特殊字符串来设置权限。Unix 具有三类权限：所有者/用户、组和其他。*所有者*类别适用于最初创建文件的用户。*组*类别适用于用户所属的任何组（以及其他用户可能所属的组）。*其他*类别适用于所有其他人。

除了这三类权限外，Unix 还具有三种主要的权限类型：*读取*文件的权限、*写入*数据到文件的权限（或删除文件），以及*执行*文件的权限（通常适用于目标代码或 Shell 脚本）。

一个典型的 chmod 选项由一个到三个字符组成，字符来自 {ugo} 集合，后面跟着一个加号或减号字符（+ 或 -，而不是 ±），再跟着一个来自 {rwx} 集合的单一字符。例如，u+r 启用用户的读取权限，u+x 启用执行权限，ugo-x 移除所有类别的执行权限。注意，ls -l 命令会列出给定文件的用户、组和其他权限。

你也可以通过指定一个三位八进制数字来设置三类权限，其中每一位代表用户（高位数字）、组（中间数字）和其他（低位数字）的 rwx 权限。例如，755 表示用户具有读/写/执行权限（111[2] = 7[8]），组和其他用户具有读和执行权限（101[2] = 5[8]）。注意，755 是你通常会赋予一个公开可用的脚本文件的典型权限。

## D.6 Shell 脚本

*Shell 脚本*是一个文本文件，bash 将其解释为一系列要执行的命令，就像在运行 bash 时每行命令是从键盘输入的一样。对于有微软 Windows 使用经验的人来说，这类似于批处理文件。本节讨论了如何使用 shell 变量和数值、使用内置的特殊 shell 变量，以及如何创建你自己的 bash Shell 脚本。

bash 解释器是一个完整的编程语言，支持条件判断和循环结构，以及命令行上命令的顺序执行。它支持 if...elif...else 语句、一个 case 语句（类似于 C 的 switch 语句），以及各种循环（如 while、for 等）。它还支持函数、本地变量以及其他通常在高级语言中发现的特性。详细讨论这些内容超出了本书的范围。有关更多信息，请参阅 D.8 节中的 Ryan’s Tutorials，第 968 页，以获取本节未涵盖的详细信息。

### D.6.1 定义 Shell 脚本变量和值

Bash 允许你定义 shell 变量。一个*shell 变量*是一个名称（类似于编程语言中的标识符），你可以为其赋值一些文本。例如，以下 bash 命令将文本 ls 赋值给 list：

```
list=ls
```

你可以通过在变量名前加上 $ 符号来告诉 bash 扩展 shell 变量名为其关联的文本。例如

```
$list
```

展开为

```
ls
```

这将显示当前目录列表。

通常情况下，你不会使用 shell 变量来创建现有命令的别名，因为 alias 命令更适合这个工作。相反，你会使用 shell 变量来跟踪路径、选项以及命令行中常用的其他信息。

Bash 提供了多个预定义的 shell 变量，包括以下内容：

$HOME    包含当前用户的主目录路径

$HOSTNAME    包含机器的名称

$PATH    包含一系列由冒号（:）分隔的目录路径，bash 在查找外部命令的可执行文件时会遍历这些路径

$PS1    包含 bash 将作为命令行提示符打印的字符串

$PWD    包含当前工作目录

有关预定义 shell 变量的完整列表，以及特别是关于 $PS1 变量的更多详细信息，请参阅 D.8 节，“更多信息”，见 第 968 页。

你为 shell 变量赋的值将在当前 bash shell 执行期间保持不变。通常，在执行 shell 脚本时，会启动一个新的 bash shell，并且在该执行过程中创建或修改的任何变量值会在该 shell 终止时丢失。为避免这个问题，可以使用内建的 export 命令：

```
export `variable name`=`value`
```

此命令将使变量赋值对父 shell 可见。通常情况下，你必须在赋值时使用 export，特别是当你希望在 shell 脚本执行完后保留变量值时。

你可以像在命令行上交互输入命令时一样，在脚本文件中定义 shell 变量。然而，如前所述，任何在 shell 脚本中定义的变量值会在 shell 终止时丢失。这是因为 bash 在启动脚本时会复制执行环境（包括所有 shell 变量的值）。你所做的任何更改或新增内容，例如创建新变量或修改现有变量，只会影响环境的副本。当脚本终止时，它会删除该副本并恢复原始环境。export 命令会告诉 bash 将变量赋值导出到父环境（同时适用于当前的本地环境）。

分配给 shell 变量的值通常被当作文本处理。因为 bash 解释器通过空格或其他分隔符来分割命令行，所以你为脚本变量分配的文本值必须由一个单一的*单词*（即被分隔符包围的字符序列）组成。如果你希望在值中包含分隔符（以及其他）字符，必须使用引号或撇号将文本值括起来，如以下示例所示：

```
value1="`Value containing delimiters (spaces)`"
value2='`Another value with delimiters`'
```

Bash 会扩展双引号（"）内的文本，并且会保持单引号（'）内的文本不变。考虑以下示例：

```
aVariable="`Some text`"
value3="aVariable=$aVariable"
value4='aVariable=$aVariable'
echo $value3
echo $value4
```

执行此序列将产生以下输出：

```
aVariable=Some Text
aVariable=$aVariable
```

Bash 会在双引号内扩展 $aVariable，但不会在单引号内扩展它。

你可能会看到被重音符号（`）包围的字符串，在 Unix 中通常被称为*反引号*。最初，这样的字符串用于包围一个命令，shell 会执行该命令，然后将程序的文本输出替换回反引号包围的字符串。这个语法在现代的 shell 中已被弃用。要捕获命令的输出并将其赋值给变量，请使用 $(command)，如以下示例所示：

```
dirListing=$(ls)
```

这会创建一个包含当前工作目录列表的字符串，并将该字符串赋值给变量 dirListing。### D.6.2 定义特殊 Shell 变量

除了脚本从父环境继承的 shell 变量外，bash 还定义了某些可能在你编写的 shell 脚本中有用的特殊 shell 变量。这些特殊变量以 $ 开头，通常与传递给脚本的命令行参数相关（参见 表 D-1）。

表 D-1：特殊 Shell 变量

| 变量 | 描述 |
| --- | --- |
| $0 | 扩展为 shell 脚本文件的路径名。 |
| $1 到 $n | 扩展为第一个、第二个、...，第 n 个命令行参数。 |
| $# | 扩展为指定参数个数的十进制数字。 |
| $* | 扩展为包含所有命令行参数的字符串。通常用于将参数传递给另一个命令。要将此命令行参数列表字符串赋值给 shell 变量，请使用 $* 来捕获文本作为单个字符串。 |
| $@ | 类似于 $*，不过这个变体会将每个参数用引号括起来。如果原始参数可能已经被引用，并且可能包含空格或其他分隔符字符，这种方式特别有用。最好也以 $@ 的形式调用它。 |

有关这些功能的更多详细信息，请参见第 D.8 节“更多信息”中的参考资料，第 968 页。

### D.6.3 编写你自己的 Shell 脚本

考虑以下来自名为*lsPix*的文件中的文本：

```
cd $HOME/Pictures
ls
```

如果你通过以下命令执行这个 shell 脚本，bash 会切换到用户主目录下的*Pictures*子目录，显示目录列表，然后将控制权返回给 bash：

```
bash lsPix
```

如果你常常执行某些 shell 脚本，在脚本前加上 bash 执行它可能会变得令人烦恼。幸运的是，Unix（通过 sh、bash 或 zsh 等 shell）提供了一种机制，可以直接将 shell 脚本作为命令来执行：使脚本文件可执行。你可以使用 chmod 命令来完成这个操作：

```
chmod 755 lsPix
```

这会将所有者的权限设置为 RWX（可读、可写和可执行），将组成员和其他用户的权限设置为 R-X（可读和可执行）。

请注意，*build* 脚本（本书中使用的）已通过 chmod 777 build 命令使其可执行（这允许所有人修改文件）。因此，你只需在命令行开头输入 ./build，而不是 bash build。

在使 bash shell 脚本可执行时，还需要在脚本文件的开头添加以下语句：

```
#! /bin/bash
```

*shebang*（#!）序列告诉 bash 这是一个 shell 脚本，并提供要执行此命令的 shell 解释器的路径。（在这种情况下，解释器将是 bash，但如果你想的话，也可以指定其他的 shell 解释器，如 /bin/sh。）如果你不知道 bash 解释器的路径，可以执行 Unix 命令 which bash 来打印所需的路径。在第一行包含 shebang 还允许文件 lspix 命令将文件识别为 shell 脚本，而不是简单的 ASCII 文本文件。

一旦你将这一行添加到*lsPix*并使文件可执行，你只需在命令行中输入以下内容来执行脚本：

```
./lsPix
```

# 字符通常用于在 shell 脚本中创建注释。除了第一行的 shebang 外，bash 解释器会忽略从 # 符号到行尾的所有文本。

重要的是要理解，shell 脚本在它们自己的 bash 副本中执行。因此，它们对 bash 环境所做的任何更改——例如使用 cd 命令设置当前工作目录或更改 shell 变量的值——在脚本终止时都会丢失。例如，当 *lsPix* 脚本终止时，当前工作目录将返回到原始目录；它不会是 *$HOME/Pictures*（除非在执行 *lsPix* 之前它就已经是 *$HOME/Pictures*）。

## D.7 构建脚本

Shell 脚本对于自动化手动操作非常有用。例如，本书使用 *build* 脚本来组装/编译大部分示例程序。以下是 *build* 脚本，并将描述其工作原理；我将逐节解释完整的脚本。

像任何好的 shell 脚本一样，构建脚本以定义要使用的 shell 解释器（此例中为 bash）的 shebang 开头：

```
#!/bin/bash
#
# build
#
# Automatically builds an Art of ARM Assembly
# example program from the command line.
#
# Usage:
#
#   build {options} fileName
#
# (no suffix on the filename.)
#
# options:
#
#   -c: Assemble .S file to object code only.
#   -pie: On Linux, generate a PIE executable.
fileName=""
compileOnly=" "
pie="-no-pie"
cFile="c.cpp"
lib=" "
```

脚本还定义了几个变量（filename、compileOnly、pie、cFile 和 lib），它们将用于在汇编和编译源文件时指定 GCC 命令行选项。

脚本的下一个部分处理构建命令行中的命令行参数：

```
❶ while [[$# -gt 0]]
do
    key="$1"
    case $key in
        -c)
        compileOnly='-c'
      ❷ shift
        ;;
        -pie)
        pie='-pie'
        shift
        ;;
        -math)
        math='-lm'
        shift
        ;;
        *)
        fileName="$1"
        shift
        ;;
    esac
done
```

while 循环逐个处理每个命令行参数 ❶。布尔表达式 $# -gt 0 会在有一个或多个命令行参数时返回 true（$# 是参数的数量）。

循环体将关键的局部变量设置为第一个命令行参数（$1）的值。接着它执行一个 case 语句，将该参数与选项 -c、-pie 和 -math 进行比较。如果参数匹配其中之一，脚本会将适当的局部变量设置为表示这些选项存在的值。如果 case 表达式不匹配任何选项，默认的 case (*) 会将文件名变量设置为命令行参数的值。

在每个 case 语句结束时，你会注意到一个 shift 语句 ❷。这个语句将所有命令行参数向左移一位（删除原来的 $1 参数），将 $1 = $2、$2 = $3，依此类推，并将参数计数 ($#) 减少 1。这为 while 循环准备了下一次迭代，处理剩下的命令行参数。

下一个部分设置了脚本中作为 gcc 命令行的一部分扩展的 objectFile 变量：

```
# If -c option was provided, assemble only the .S
# file and produce a .o output file.
#
# If -c not specified, compile both c.cpp and the .S
# file and produce an executable:
if ["$compileOnly" = '-c']; then
    objectFile="-o $fileName".o
    cFile=" "
else
    objectFile="-o $fileName"
fi
```

这段代码将 objectFile 设置为一个字符串，该字符串会在 gcc 命令行中指定一个目标文件名。如果没有 -c 选项，该代码会将 cFile 设置为空字符串，这样 gcc 命令就不会编译 *c.cpp*（默认情况）。

以下 *build* 脚本部分会删除该命令可能创建的任何现有目标文件或可执行文件：

```
# If the executable already exists, delete it:
if test -e "$fileName"; then
    rm "$fileName"
fi
# If the object file already exists, delete it:
if test -e "$fileName".o; then
    rm "$fileName".o
fi
```

test 内建函数在指定文件存在时返回 true。因此，如果对象文件和可执行文件已经存在，这些 if 语句将删除它们。

接下来，*aoaa.inc* 头文件需要定义 isLinux 或 isMacOS 符号，以便确定操作系统。定义这些符号后，*aoaa.inc* 能够选择操作系统特定的代码，从而使示例代码能够在这两个操作系统上（便捷地）编译。为了避免强制用户手动定义该符号，*build* 脚本在调用 GCC 时会自动定义其中一个符号。为此，*build* 使用 uname 命令，它返回操作系统内核的名称：

```
# Determine what OS you're running under (Linux or Darwin [macOS]) and
# issue the appropriate GCC command to compile/assemble the files:
unamestr=$(uname)
```

在 Linux 下，uname 返回字符串 Linux；在 macOS 下，它返回字符串 Darwin。

最后，*build* 脚本根据操作系统的适当命令行参数调用 GCC 编译器：

```
if ["$unamestr" = 'Linux']; then
    gcc -D isLinux=1 $pie $compileOnly $objectFile  $cFile $fileName.S $math
elif ["$unamestr" = 'Darwin']; then
    gcc -D isMacOS=1  $compileOnly $objectFile $cFile  $fileName.S -lSystem $math
fi
```

注意命令行选项 -D name=1，它根据需要定义 isLinux 或 isMacOS 符号。还要注意，pie（位置无关代码）选项仅在 Linux 下编译时出现，因为 macOS 的代码总是位置无关的。

如果你有需求，修改 *build* 脚本以添加更多功能是非常容易的。例如，脚本的一个限制是它只允许你指定单个汇编语言源文件（如果指定两个或更多名称，它只会使用你指定的最后一个名称）。你可以通过在 case 语句中的三处修改来改变这一点。

第一个修改是将文件名附加并为 fileName 变量添加 .S 后缀，而不是替换其值。你还必须将可执行输出文件名设置为命令行上指定的第一个汇编文件：

```
*)
    if[fileName = ""]
    then
        objectFile = "$1"
    fi
    fileName="$filename $1.S"
    shift
    ;;
```

接下来的修改是在指定编译模式时，将 objectFile 设置为空字符串：

```
if ["$compileOnly" = '-c']; then
cFile=" "
else
    objectFile="-o $fileName"
fi
```

原始代码将其设置为指定的文件名；然而，这在仅编译模式下是默认设置，而在汇编多个源文件时，指定单个对象名称会存在问题。

最后的修改是修改两个 gcc 命令行，将汇编文件名中的 .S 后缀去掉（因为这已经在 case 语句中添加了）：

```
if ["$unamestr" = 'Linux']; then
    gcc -D isLinux=1 $pie $compileOnly $objectFile  $cFile $fileName $math
elif ["$unamestr" = 'Darwin']; then
    gcc -D isMacOS=1  $compileOnly $objectFile $cFile  $fileName -lSystem $math
fi
```

然而，如果你打算使用多个源文件进行复杂的汇编，可能还是使用 makefile 比使用 shell 脚本更合适。

## D.8 更多信息

+   有关 Unix 正则表达式和通配符的详细信息，请参见 Bash 参考手册 *[`<wbr>www<wbr>.gnu<wbr>.org<wbr>/savannah<wbr>-checkouts<wbr>/gnu<wbr>/bash<wbr>/manual<wbr>/bash<wbr>.html`](https://www.gnu.org/savannah-checkouts/gnu/bash/manual/bash.html)*。

+   有关 bash shell 脚本的更多信息，请查阅 William Shotts 编写的 *The Linux Command Line*，第二版（No Starch Press，2019 年）。

+   关于在执行路径中包含 ./ 的风险，请参见 Unix & Linux Stack Exchange 上的问题 *[`<wbr>unix<wbr>.stackexchange<wbr>.com<wbr>/questions<wbr>/65700<wbr>/is<wbr>-it<wbr>-safe<wbr>-to<wbr>-add<wbr>-to<wbr>-my<wbr>-path<wbr>-how<wbr>-come`](https://unix.stackexchange.com/questions/65700/is-it-safe-to-add-to-my-path-how-come)*。

+   bash shell 脚本变量的完整列表可见于《高级 Bash 脚本编程指南》网站，*[`tldp.org/LDP/abs/html/internalvariables.html`](https://tldp.org/LDP/abs/html/internalvariables.html)*。

+   有关更改命令行提示符的详细信息，请参阅 phoenixNap 网站，*[`phoenixnap.com/kb/change-bash-prompt-linux`](https://phoenixnap.com/kb/change-bash-prompt-linux)*。

以下是一些描述如何编写 bash 脚本的网站：

+   freeCodeCamp: *[`www.freecodecamp.org/news/shell-scripting-crash-course-how-to-write-bash-scripts-in-linux`](https://www.freecodecamp.org/news/shell-scripting-crash-course-how-to-write-bash-scripts-in-linux)*

+   Ryan’s Tutorials: *[`ryanstutorials.net/bash-scripting-tutorial/bash-script.php`](https://ryanstutorials.net/bash-scripting-tutorial/bash-script.php)*

+   Linux Hint: *[`linuxhint.com`](https://linuxhint.com)*

+   Bash 脚本备忘单: *[`devhints.io/bash`](https://devhints.io/bash)*
