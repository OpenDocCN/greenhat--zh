

# 第二章：2 流程控制与文本处理



![](img/opener.jpg)

本章介绍了可以使脚本更智能的 Bash 概念。你将学习如何测试条件、使用循环、将代码整合为函数、将命令发送到后台等。你还将学习一些定制 Bash 环境的方法，适用于渗透测试。

## 测试操作符

Bash 让我们在满足某些特定条件时有选择地执行命令。我们可以使用 *测试操作符* 来构造各种条件，比如检查一个值是否等于另一个值，文件是否为某种类型，或者一个值是否大于另一个值。我们经常依赖这些测试来决定是否继续执行一段代码，因此构造这些测试是 Bash 编程的基础。

Bash 有多种类型的测试操作符。*文件测试操作符* 允许我们对文件系统中的文件进行测试，例如检查文件是否可执行或某个目录是否存在。表 2-1 显示了可用测试的简短列表。

表 2-1：文件测试操作符

| 操作符 | 描述 |
| --- | --- |
| -d | 检查文件是否为目录 |
| -r | 检查文件是否可读 |
| -x | 检查文件是否可执行 |
| -w | 检查文件是否可写 |
| -f | 检查文件是否为常规文件 |
| -s | 检查文件大小是否大于零 |

你可以在 *[`ss64.com/bash/test.html`](https://ss64.com/bash/test.html)* 找到完整的文件测试操作符列表，或者通过运行 `man test` 命令来查看。

*字符串比较操作符* 允许我们进行与字符串相关的测试，例如检查一个字符串是否等于另一个字符串。表 2-2 显示了字符串比较操作符。

表 2-2：字符串比较操作符

| 操作符 | 描述 |
| --- | --- |
| = | 检查一个字符串是否等于另一个字符串 |
| == | 在 [[]] 构造中，= 的同义词 |
| != | 检查一个字符串是否不等于另一个字符串 |
| < | 检查一个字符串是否在另一个字符串之前（按字母顺序） |
| > | 检查一个字符串是否在另一个字符串之后（按字母顺序） |
| -z | 检查字符串是否为空 |
| -n | 检查字符串是否非空 |

*整数比较操作符* 允许我们对整数进行检查，例如检查一个整数是否小于或大于另一个整数。表 2-3 显示了可用的操作符。

表 2-3：整数比较操作符

| 操作符 | 描述 |
| --- | --- |
| -eq | 检查一个数字是否等于另一个数字 |
| -ne | 检查一个数字是否不等于另一个数字 |
| -ge | 检查一个数字是否大于或等于另一个数字 |
| -gt | 检查一个数字是否大于另一个数字 |
| -lt | 检查一个数字是否小于另一个数字 |
| -le | 检查一个数是否小于或等于另一个数 |

让我们在流控制机制中使用这些操作符，以决定接下来要运行哪些代码。

## if 条件

在 bash 中，我们可以使用 if 条件来仅在满足特定条件时执行代码。清单 2-1 展示了它的语法。

```
if [[`condition`]]; then
  # Do something if the condition is met.
else
  # Do something if the condition is not met.
fi 
```

清单 2-1：if 语句的结构

我们以 if 关键字开始，接着是在双方括号（[[]]）之间的测试条件。然后使用;字符将 if 关键字与 then 关键字分开，这允许我们引入一个仅在条件满足时运行的代码块。

接下来，我们使用 else 关键字引入一个回退代码块，当条件不满足时运行。请注意，else 是可选的，有时候你可能不需要它。最后，我们使用 fi 关键字（这是 if 的反义词）来关闭 if 条件。

> 注意

*在某些操作系统中（如经常用于容器的操作系统），默认 shell 可能不一定是 bash。为了考虑这些情况，您可能希望使用单方括号（* [...] *）而不是双方括号来包含您的条件。单方括号的使用符合可移植操作系统接口标准，并且几乎可以在包括 Linux 在内的任何 Unix 衍生系统上运行。*

让我们看看 if 条件在实践中的运用。清单 2-2 使用 if 条件来测试文件是否存在，如果不存在则创建它。

test_if_file _exists.sh

```
#!/bin/bash
FILENAME="flow_control_with_if.txt"

if [[-f "${FILENAME}"]]; then
 echo "${FILENAME} already exists."
  exit 1
else
  touch "${FILENAME}"
fi 
```

清单 2-2：用于检查文件是否存在的 if 条件

我们首先创建一个名为 FILENAME 的变量，其中包含我们需要的文件名。这样可以避免在代码中重复文件名。然后，我们引入 if 语句，其中包含使用-f 文件测试操作符来测试文件是否存在的条件。如果这个条件为真，则使用 echo 打印一条消息到屏幕上，解释文件已经存在，并使用状态码 1（失败）退出程序。在 else 块中，只有当文件不存在时才会执行，我们使用 touch 命令创建文件。

> 注意

*你可以从* [`github.com/dolevf/Black-Hat-Bash/blob/master/ch02`](https://github.com/dolevf/Black-Hat-Bash/blob/master/ch02) *下载本章的脚本。*

保存文件并执行它。当您运行 ls 时，您应该在当前目录中看到*flow_control_with_if.txt*文件。

清单 2-3 展示了实现相同结果的另一种方法：它使用非运算符（!）来检查目录*不存在*，如果不存在则创建它。这个示例代码行数更少，并且完全消除了 else 块的需要。

```
#!/bin/bash
FILENAME="flow_control_with_if.txt"

if [[! -f "${FILENAME}"]]; then
  touch "${FILENAME}"
fi 
```

清单 2-3：使用否定检查来测试文件是否存在

让我们探讨使用我们已经涵盖的其他类型测试操作符的 if 条件。清单 2-4 展示了一个字符串比较测试。它使用等于运算符（==）进行字符串比较，测试两个变量是否相等。

string _comparison.sh

```
#!/bin/bash
VARIABLE_ONE="nostarch"
VARIABLE_TWO="nostarch"

if [["${VARIABLE_ONE}" == "${VARIABLE_TWO}"]]; then
  echo "They are equal!"
else
  echo "They are not equal!"
fi 
```

图 2-4：比较两个字符串变量

此脚本将比较两个值都为 nostarch 的变量，并使用 echo 命令打印出 They are equal!。

接下来是一个整数比较测试，它接受两个整数并检查哪一个是较大的数（图 2-5）。

integer _comparison.sh

```
#!/bin/bash
VARIABLE_ONE="10"
VARIABLE_TWO="20"

if [["${VARIABLE_ONE}" -gt "${VARIABLE_TWO}"]]; then
  echo "${VARIABLE_ONE} is greater than ${VARIABLE_TWO}."
else
  echo "${VARIABLE_ONE} is less than ${VARIABLE_TWO}."
fi 
```

图 2-5：比较整数

我们创建了两个变量，VARIABLE_ONE 和 VARIABLE_TWO，并分别赋值为 10 和 20。然后我们使用 -gt 运算符比较这两个值，并基于整数比较打印结果。

### 链接条件

到目前为止，我们使用 if 来检查单个条件是否满足。但与大多数编程语言一样，我们也可以使用 OR（||）和 AND（&&）运算符同时检查多个条件。

例如，如果我们想要检查文件是否存在并且其大小大于零，图 2-6 就是这样做的。

```
#!/bin/bash

echo "Hello World!" > file.txt

if [[-f "file.txt"]] && [[-s "file.txt"]]; then
  echo "The file exists and its size is greater than zero."
fi 
```

图 2-6：使用 AND 链接两个文件测试条件

此代码向文件写入内容，然后检查该文件是否存在以及其大小是否大于零。只有在这两个条件均满足时，echo 命令才会执行。如果任一条件返回 false，则什么也不会发生。

要演示 OR 条件，图 2-7 检查一个变量是文件还是目录。

```
#!/bin/bash
DIR_NAME="dir_test"

mkdir "${DIR_NAME}"

if [[-f "${DIR_NAME}"]] || [[-d "${DIR_NAME}"]]; then
 echo "${DIR_NAME} is either a file or a directory."
fi 
```

图 2-7：使用 OR 链接两个文件测试条件

此代码首先创建一个目录，然后使用 OR（||）运算符的 if 条件检查变量是文件（-f）还是目录（-d）。第二个条件应该评估为 true，并且 echo 命令应该执行。

### 测试命令成功

我们甚至可以测试命令的退出代码，以确定它们是否成功（图 2-8）。

```
if `command`; then
  # `command` was successful.
fi

if ! `command`; then
  # `command` was unsuccessful.
fi 
```

图 2-8：基于退出代码值执行命令

在 bash 中，你经常会使用这种技术，因为命令并不保证成功。失败可能是以下原因之一：

+   创建资源时缺少必要的权限

+   尝试执行操作系统上不可用的命令

+   下载文件时磁盘空间已满

+   在执行网络工具时网络断开

要查看这种技术的工作原理，请在终端中执行以下操作：

```
$ **if touch test123; then**
 **echo "OK: file created"**
 **fi**

OK: file created 
```

我们尝试创建一个文件。因为文件创建成功，我们打印一条消息以指示这一点。

### 检查后续条件

如果第一个 if 条件失败，你可以使用 elif 关键字（*else if* 的简写）检查其他条件。为了展示其工作原理，让我们编写一个程序来检查传递给它的命令行参数。 图 2-9 将输出一个消息，说明参数是文件还是目录。

if_elif.sh

```
#!/bin/bash
USER_INPUT="${1}"

❶ if [[-z "${USER_INPUT}"]]; then
  echo "You must provide an argument!"
  exit 1
fi
❷ if [[-f "${USER_INPUT}"]]; then
  echo "${USER_INPUT} is a file."
❸ elif [[-d "${USER_INPUT}"]]; then
  echo "${USER_INPUT} is a directory."
else
❹ echo "${USER_INPUT} is not a file or a directory."
fi 
```

图 2-9：使用 if 和 elif 语句

我们从一个 if 语句开始，检查变量 USER_INPUT 是否为 null ❶。这允许我们在没有收到用户的命令行参数时，通过使用 exit 1 提前退出脚本。接着，我们开始第二个 if 条件，使用文件测试操作符检查输入是否为文件 ❷。在此条件下，我们使用 elif 来测试参数是否为目录 ❸。此条件只有在文件测试失败时才会被测试。如果这两个条件都不成立，脚本将回应说参数既不是文件也不是目录 ❹。

## 函数

*函数* 帮助我们重用代码块，以便避免重复编写。它们允许我们通过简单地输入函数名称来同时执行多个命令和其他 Bash 代码。要定义一个新函数，输入它的名称，后跟括号。然后将希望该函数运行的代码放在大括号中（列表 2-10）。

```
#!/bin/bash

say_name(){
  echo "Black Hat Bash"
} 
```

列表 2-10：定义一个函数

在这里，我们定义了一个名为 say_name() 的函数，它执行一个简单的 echo 命令。要调用函数，只需输入其名称：

```
say_name
```

如果函数没有被调用，它内部的命令不会执行。

### 返回值

像命令及其退出状态一样，函数也可以通过使用 return 关键字返回值。如果没有 return 语句，函数将返回其运行的最后一个命令的退出代码。例如，列表 2-11 中的函数会根据当前用户是否为 root 返回不同的值。

check_root _function.sh

```
#!/bin/bash

# This function checks if the current user ID equals zero.
❶ check_if_root(){
❷ if [["${EUID}" -eq "0"]]; then
    return 0
  else
    return 1
  fi
}

if check_if_root; then
  echo "User is root!"
else
  echo "User is not root!"
fi 
```

列表 2-11：测试函数返回真或假的 if 条件

我们定义了 check_if_root() 函数 ❶。在此函数内，我们使用一个带有整数比较测试的 if 条件 ❷，通过访问环境变量 EUID 来获取当前有效用户的 ID，并检查其是否等于 0。如果是，那么用户是 root，函数返回 0；如果不是，则返回 1。接下来，我们调用 check_if_root 函数，并检查其是否返回 0，这意味着用户是 root。否则，我们打印出用户不是 root 的信息。

执行特权操作的 Bash 脚本通常会在尝试安装软件、创建用户、删除组等操作之前检查用户是否为 root。若在 Linux 上没有必要的特权执行特权操作，则会导致错误，因此此检查有助于处理这些情况。

### 接受参数

在 第一章中，我们介绍了如何将参数传递给命令行上的命令。函数也可以使用相同的语法来接受参数。例如，列表 2-12 中的函数会打印它接收到的前三个参数。

```
#!/bin/bash

print_args(){
 echo "first: ${1}, second: ${2}, third: ${3}"
}

❶ print_args No Starch Press 
```

列表 2-12：带有参数的函数

要调用带有参数的函数，输入其名称和由空格分隔的参数 ❶。将此脚本保存为 *function_with_args.sh* 并运行：

```
$ **chmod u+x function_with_args.sh**
$ **./function_with_args.sh**

first: No, second: Starch, third: Press 
```

你应该看到类似于这里所展示的输出。

## 循环与循环控制

和许多编程语言一样，bash 通过使用 *循环* 让你重复执行代码块。循环在你的渗透测试冒险中尤其有用，因为它们可以帮助你完成如下任务：

+   在重启后持续检查某个 IP 地址是否在线，直到该 IP 地址响应

+   迭代主机名列表（例如，针对每个主机运行特定的漏洞攻击或判断是否有防火墙在保护它们）

+   测试某个条件并在满足时运行循环（例如，检查某个主机是否在线，如果在线，则对其进行暴力破解攻击）

以下部分将介绍 bash 中的三种循环（while、until 和 for），以及用于处理循环的 break 和 continue 语句。

### while

在 bash 中，while 循环允许你运行一个代码块，直到测试返回成功的退出状态代码。例如，你可能会在渗透测试中使用它们，持续对网络进行端口扫描，并捕捉加入网络的任何新主机。

清单 2-13 显示了 while 循环的语法。

```
while `some_condition`; do
  # Run commands while the condition is true.
done 
```

清单 2-13：一个 while 循环

该循环以 while 关键字开始，后面跟着描述条件的表达式。然后，我们用 do 和 done 关键字将要执行的代码包围起来，这些关键字定义了代码块的开始和结束。

你可以使用 while 循环通过将 true 作为条件来无限次运行一段代码；因为 true 总是返回成功的退出代码，所以代码将一直运行。让我们使用 while 循环重复地将命令打印到屏幕上。将 清单 2-14 保存为名为 *basic_while.sh* 的文件并运行它。

```
#!/bin/bash

while true; do
  echo "Looping..."
  sleep 2
done 
```

清单 2-14：以两秒间隔重复运行命令

你应该看到以下输出：

```
$ **chmod u+x basic_while.sh**
$ **./basic_while.sh**

Looping...
Looping...
`--snip--` 
```

接下来，让我们编写一个更复杂的 while 循环，直到它在文件系统中找到特定文件为止（清单 2-15）。使用 CTRL-C 随时停止代码执行。

while_loop.sh

```
#!/bin/bash
❶ SIGNAL_TO_STOP_FILE="stoploop"

❷ while [[! -f "${SIGNAL_TO_STOP_FILE}"]]; do
  echo "The file ${SIGNAL_TO_STOP_FILE} does not yet exist..."
  echo "Checking again in 2 seconds..."
  sleep 2
done

❸ echo "File was found! Exiting..." 
```

清单 2-15：文件监控

在 ❶ 处，我们定义了一个变量，表示 while 循环 ❷ 检查的文件名，并使用文件测试操作符。直到条件满足，循环才会退出。一旦文件可用，循环将停止，脚本将继续执行 echo 命令 ❸。将此文件保存为 *while_loop.sh* 并运行：

```
$ **chmod u+x while_loop.sh**
$ **./while_loop.sh**

The file stoploop does not yet exist...
Checking again in 2 seconds...
`--snip--` 
```

在脚本运行时，在与脚本相同的目录中打开第二个终端，并创建 *stoploop* 文件：

```
$ **touch stoploop**
```

完成后，你应该看到脚本跳出循环并打印以下内容：

```
File was found! Exiting...
```

我们可以使用 while 循环来监视文件系统事件，例如文件创建或删除，或者进程启动时。这在应用程序存在只能暂时利用的漏洞时非常有用。例如，考虑一个每天在特定时间运行的应用程序，它检查文件*/tmp/update.sh*是否存在；如果存在，应用程序将以*root*用户身份执行该文件。使用 while 循环，我们可以监视该应用程序的启动，然后及时创建该文件，以便我们的命令能被该应用程序执行。

### until

而 while 在条件成功时持续运行，until 在条件失败时持续运行。清单 2-16 展示了 until 循环的语法。

```
until `some_condition`; do
  # Run some commands until the condition is no longer false.
done 
```

清单 2-16：一个 until 循环

清单 2-17 使用 until 来运行一些命令，直到文件大小大于零（意味着它不再为空）。

until_loop.sh

```
#!/bin/bash
FILE="output.txt"

touch "${FILE}"
until [[-s "${FILE}"]]; do
  echo "${FILE} is empty..."
  echo "Checking again in 2 seconds..."
  sleep 2
done

echo "${FILE} appears to have some content in it!" 
```

清单 2-17：检查文件的大小

我们首先创建一个空文件，然后开始一个循环，直到文件不再为空。在循环内部，我们将消息打印到终端。将此文件保存为*until_loop.sh*并运行：

```
$ **chmod u+x until_loop.sh**
$ **./until_loop.sh**

output.txt is empty...
Checking again in 2 seconds...
`--snip--` 
```

此时，脚本已经创建了文件*output.txt*，但它是一个空文件。我们可以使用 du（磁盘使用情况）命令来检查这一点：

```
$ **du -sb output.txt**
0       output.txt 
```

打开另一个终端并导航到脚本保存的位置，然后向文件追加一些内容，使其大小不再为零：

```
$ **echo "until_loop_will_now_stop!" > output.txt**
```

脚本应退出循环，您应该看到它打印以下内容：

```
output.txt appears to have some content in it!
```

### for

for 循环会遍历一个*序列*，例如文件名或变量的列表，甚至是通过运行命令生成的一组值。在 for 循环内部，我们定义一组命令，这些命令会对列表中的每个值执行，每个值会被赋给我们定义的变量名。

清单 2-18 展示了 for 循环的语法。

```
for `variable_name` in `LIST`; do
  # Run some commands for each item in the sequence.
done 
```

清单 2-18：一个 for 循环

使用 for 循环的一种简单方法是多次执行相同的命令。例如，清单 2-19 打印从 1 到 10 的数字。

```
#!/bin/bash

for index in $(seq 1 10); do
  echo "${index}"
done 
```

清单 2-19：在 for 循环中计数到 10

保存并运行此脚本。您应该看到以下输出：

```
1
2
3
4
5
6
7
8
9
10 
```

一个更实际的例子可能是使用 for 循环对传递到命令行的一组 IP 地址运行命令。清单 2-20 获取传递给脚本的所有参数，然后遍历它们并为每个参数打印一条消息。

```
#!/bin/bash

for ip_address in "$@"; do
  echo "Taking some action on IP address ${ip_address}"
done 
```

清单 2-20：遍历命令行参数

将此脚本保存为*for_loop_arguments.sh*并按以下方式运行：

```
$ **chmod u+x for_loop_arguments.sh**
$ **./for_loop_arguments.sh 10.0.0.1 10.0.0.2 192.168.1.1 192.168.1.2**

Taking some action on IP address 10.0.0.1
Taking some action on IP address 10.0.0.2
`--snip--` 
```

我们甚至可以对命令的输出运行 for 循环，例如 ls。在清单 2-21 中，我们打印当前工作目录中所有文件的名称。

```
#!/bin/bash

for file in $(ls .); do
  echo "File: ${file}"
done 
```

清单 2-21：遍历当前目录中的文件

我们使用 for 循环迭代 ls . 命令的输出，后者列出当前目录中的文件。每个文件会作为 for 循环的一部分被赋值给 file 变量，因此我们可以使用 echo 来打印它的名称。例如，如果我们想对目录中的所有文件进行批量上传或重命名，这种技术将会很有用。

### break 和 continue

循环可以无限运行，或者直到满足某个条件为止。但我们也可以通过使用 break 关键字在任何时刻退出循环。这个关键字提供了一种替代 exit 命令的方式，后者会导致整个脚本退出，而不仅仅是循环。使用 break，我们可以离开循环并进入下一个代码块（Listing 2-22）。

```
#!/bin/bash

while true; do
  echo "in the loop"
  break
done

echo "This code block will be reached." 
```

Listing 2-22：从循环中跳出

在这种情况下，最后的 echo 命令将被执行。

continue 语句用于跳转到循环的下一次迭代。我们可以用它跳过序列中的某个值。为了说明这一点，下面我们创建三个空文件，以便我们能对它们进行迭代：

```
$ **touch example_file1 example_file2 example_file3**
```

接下来，我们的 for 循环将向每个文件写入内容，跳过第一个文件 *example_file1*，这个文件会被留空（Listing 2-23）。

```
#!/bin/bash

❶ for file in example_file*; do
  if [["${file}" == "example_file1"]]; then
    echo "Skipping the first file"
  ❷ continue
  fi

  echo "${RANDOM}" > "${file}"
done 
```

Listing 2-23：在 for 循环中跳过元素

我们从 example_file* 通配符开始一个 for 循环，它将扩展为匹配脚本运行的目录中所有以 *example_file* 开头的文件 ❶。因此，循环应该会遍历我们之前创建的所有三个文件。在循环内，我们使用字符串比较来检查文件名是否等于 *example_file1*，因为我们想跳过这个文件，不对它进行任何修改。如果条件成立，我们使用 continue 语句 ❷ 继续到下一次迭代，保持该文件不变。然后，在循环的后续部分，我们使用 echo 命令和环境变量 ${RANDOM} 生成一个随机数并将其写入文件。

将这个脚本保存为 *for_loop_continue.sh*，并在与这三个文件相同的目录下执行：

```
$ **chmod u+x for_loop_continue.sh**
$ **./for_loop_continue.sh**

Skipping the first file 
```

如果你检查这些文件，你应该看到第一个文件是空的，而其他两个文件包含一个随机数，这是因为脚本将 ${RANDOM} 环境变量的值回显到它们中。

## case 语句

在 bash 中，case 语句允许你以更简洁的方式测试多个条件，通过使用更易读的语法。通常，它们帮助你避免使用许多 if 语句，这些语句随着代码量的增加，可能变得更难阅读。

Listing 2-24 展示了 case 语句的语法。

```
case `EXPRESSION` in
  `PATTERN1`)
    # Do something if the first condition is met.
  ;;
  `PATTERN2`)
    # Do something if the second condition is met.
  ;;
esac 
```

Listing 2-24：一个 case 语句

case 语句以关键字 case 开头，后跟一个表达式，比如你希望匹配模式的变量。此示例中的 PATTERN1 和 PATTERN2 代表你希望与表达式进行比较的模式（例如正则表达式、字符串或整数）。要结束一个 case 语句，使用关键字 esac（case 的反转）。

让我们看一个示例 case 语句，该语句检查 IP 地址是否存在于特定的私有网络中（列表 2-25）。

case_ip_address_check.sh

```
#!/bin/bash
IP_ADDRESS="${1}"

case ${IP_ADDRESS} in
  192.168.*)
    echo "Network is 192.168.x.x"
  ;;
  10.0.*)
    echo "Network is 10.0.x.x"
  ;;
  *)
 echo "Could not identify the network"
  ;;
esac 
```

列表 2-25：检查 IP 地址并确定其网络

我们定义一个变量，期望传入一个命令行参数（${1}），并将其保存到 IP_ADDRESS 变量中。然后使用一个模式检查 IP_ADDRESS 变量是否以 192.168. 开头，第二个模式检查它是否以 10.0. 开头。

我们还定义了一个默认的通配符模式 *，如果没有其他模式匹配，它会向用户返回默认信息。

将此文件保存为 *case_ip_address_check.sh* 并运行：

```
$ **chmod u+x case_ip_address_check.sh**
$ **./case_ip_address_check.sh 192.168.12.55**
Network is 192.168.x.x

$ **./case_ip_address_check.sh 212.199.2.2**
Could not identify the network 
```

case 语句可用于各种应用场景。例如，它可以根据用户输入的内容运行不同的函数。使用 case 语句是一种很好的方法，可以在不牺牲代码可读性的情况下处理多个条件的评估。

## 文本处理与解析

在 bash 中，你最常做的事情之一就是处理文本。你可以通过运行一次性的命令来解析文本，或者使用脚本将解析后的数据存储在一个变量中，以便后续使用。这两种方法对于许多场景都很重要。

若要自行测试本节中的命令，请从 *[`github.com/dolevf/Black-Hat-Bash/blob/master/ch02/log.txt`](https://github.com/dolevf/Black-Hat-Bash/blob/master/ch02/log.txt)* 下载示例日志文件。此文件使用空格分隔，每个段落代表特定的数据类型，如客户端的源 IP 地址、时间戳、超文本传输协议（HTTP）方法、HTTP 路径、HTTP 用户代理字段、HTTP 状态码等。

### 使用 grep 过滤

grep 命令是当前最流行的 Linux 命令之一。我们使用 grep 从数据流中过滤出感兴趣的信息。在最基本的形式中，我们可以像列表 2-26 中展示的那样使用它。

```
$ **grep "35.237.4.214" log.txt**
```

列表 2-26：从文件中筛选特定字符串

此 grep 命令将读取文件并提取包含 IP 地址 35.237.4.214 的所有行。

我们甚至可以同时使用 grep 来匹配多个模式。以下的反斜杠管道符号（\|）作为“或”条件：

```
$ **grep "35.237.4.214\|13.66.139.0" log.txt**
```

或者，我们可以使用多个 grep 模式与 -e 参数来完成相同的操作：

```
$ **grep -e "35.237.4.214" -e "13.66.139.0" log.txt**
```

如你在第一章中学到的，我们可以使用管道（|）命令将一个命令的输出作为另一个命令的输入。在以下示例中，我们运行 ps 命令并使用 grep 来筛选特定的行。ps 命令列出了系统中的进程：

```
$ **ps | grep TTY**
```

默认情况下，grep 是区分大小写的。我们可以通过使用 -i 标志使搜索不区分大小写：

```
$ **ps | grep -i tty**
```

我们还可以使用 grep 的 -v 参数来排除包含某个模式的行：

```
$ **grep -v "35.237.4.214" log.txt**
```

若只打印匹配的模式，而不打印整个包含该模式的行，请使用 -o：

```
$ **grep -o "35.237.4.214" log.txt**
```

该命令还支持正则表达式、锚定、分组等功能。使用 man grep 命令可以了解更多关于它的功能。

### 使用 awk 进行过滤

awk 命令是一个数据处理和提取的瑞士军刀。你可以用它从文件中识别并返回特定的字段。要看看 awk 是如何工作的，可以再次仔细查看我们的日志文件。如果我们需要从文件中打印出所有 IP 地址呢？这可以通过 awk 轻松完成：

```
$ **awk '{print $1}' log.txt**
```

$1 表示文件中每一行的第一个字段，其中包含 IP 地址。默认情况下，awk 将空格或制表符视为分隔符或定界符。

使用相同的语法，我们可以打印其他字段，比如时间戳。以下命令过滤文件中每一行的前三个字段：

```
$ **awk '{print $1,$2,$3}' log.txt**
```

使用类似的语法，我们可以同时打印第一个和最后一个字段。在这种情况下，NF 表示最后一个字段：

```
$ **awk '{print $1,$NF}' log.txt**
```

我们还可以更改默认的分隔符。例如，如果我们有一个由逗号分隔的文件（即 CSV 文件，逗号分隔值文件），而不是由空格或制表符分隔，我们可以通过传递-F 标志给 awk 来指定分隔符的类型：

```
$ **awk -F',' '{print $1}' example_csv.txt**
```

我们甚至可以使用 awk 打印文件的前 10 行。这模仿了 Linux 命令 head 的行为；NR 表示记录的总数，并且是 awk 内建的：

```
$ **awk 'NR < 10' log.txt**
```

你会发现将 grep 和 awk 结合使用非常有用。例如，你可能想先找到文件中包含 IP 地址 42.236.10.117 的行，然后打印该 IP 请求的 HTTP 路径：

```
$ **grep "42.236.10.117" log.txt | awk '{print $7}'**
```

awk 命令是一个功能强大的工具，我们鼓励你通过运行 man awk 进一步深入了解它的功能。

### 使用 sed 编辑流

sed（流编辑器）命令对文本执行操作。例如，它可以替换文件中的文本，修改命令输出中的文本，甚至删除文件中的特定行。

让我们使用 sed 将文件*log.txt*中任何提到*Mozilla*的地方替换为*Godzilla*。我们使用它的 s（替换）命令和 g（全局）命令，以便在整个文件中进行替换，而不仅仅是第一个出现的位置：

```
$ **sed 's/Mozilla/Godzilla/g' log.txt**
```

这将输出文件的修改版本，但不会改变原始版本。你可以将输出重定向到一个新文件来保存更改：

```
$ **sed 's/Mozilla/Godzilla/g' log.txt > newlog.txt**
```

我们还可以使用 sed 通过/ //语法删除文件中的任何空白字符，这将把空白字符替换为空值，完全从输出中去除它们：

```
$ **sed 's/ //g' log.txt**
```

如果你需要删除文件的行，可以使用 d 命令。在以下命令中，1d 删除（d）第 1 行（1）：

```
$ **sed '1d' log.txt**
```

要删除文件的最后一行，可以使用美元符号（$），它表示最后一行，配合 d 命令：

```
$ **sed '$d' log.txt**
```

你还可以删除多行，比如第 5 行和第 7 行：

```
$ **sed '5,7d' log.txt**
```

最后，你可以打印（p）特定的行范围，比如第 2 行到第 15 行：

```
$ **sed -n '2,15 p' log.txt**
```

当你传递-i 参数给 sed 时，它会直接修改文件本身，而不是创建一个修改过的副本：

```
$ **sed -i '1d' log.txt**
```

这个强大的工具还可以做更多的事情。使用 `man sed` 命令可以找到更多使用 `sed` 的方法。

## 作业控制

当你逐渐精通 bash 时，你会开始编写需要一个小时完成的复杂脚本，或者需要持续运行的脚本。并非所有脚本都需要在前台执行并阻塞其他命令的执行。相反，你可能希望将某些脚本作为后台作业运行，无论是因为它们需要较长时间才能完成，还是因为它们的运行输出不重要，你只关心最终结果。

你在终端中运行的命令会占用该终端，直到命令执行完毕。这些命令被认为是 *前台作业*。在第一章中，我们使用了符号 `&` 将命令发送到后台。这样，命令就变成了 *后台作业*，允许我们不阻塞其他命令的执行。

### 后台与前台管理

为了练习管理后台和前台作业，我们直接在终端运行一个命令并将其发送到后台：

```
$ **sleep 100 &**
```

请注意，在这个 `sleep` 命令运行 100 秒的过程中，我们仍然可以继续在终端上工作。我们可以使用 `ps` 命令验证这个已启动的进程是否在运行：

```
$ **ps -ef | grep sleep**

user    1827    1752 cons0    19:02:29 /usr/bin/sleep 
```

现在，这个作业已经在后台运行，我们可以使用 `jobs` 命令查看当前正在运行的作业：

```
$ **jobs**

[1]+  Running                 sleep 100 & 
```

输出显示 `sleep` 命令处于运行状态，且其作业 ID 为 1。

我们可以通过执行 `fg` 命令并指定作业 ID 将作业从后台迁移到前台：

```
$ **fg %1**

sleep 100 
```

此时，`sleep` 命令正在占用终端，因为它在前台运行。你可以按下 CTRL-Z 来挂起该进程，这将在作业表中显示如下输出：

```
[1]+  Stopped                 sleep 100
```

若要将该任务再次以运行状态发送到后台，使用 `bg` 命令并指定作业 ID：

```
$ **bg %1**

[1]+ sleep 100 & 
```

这里，我们提供了作业 ID 为 1。

### 登出后保持作业运行

无论你是将作业发送到后台，还是在前台运行作业，如果关闭终端或登出，进程都无法继续存活。如果关闭终端，进程将收到一个 SIGHUP 信号并终止。

如果我们希望在退出终端窗口或关闭终端后仍然保持脚本在后台运行该怎么办？为了实现这一点，我们可以在脚本或命令前添加 `nohup`（无挂断）命令：

```
$ **nohup ./my_script.sh &**
```

`nohup` 命令会创建一个名为 *nohup.out* 的文件，存储标准输出流数据。如果你不希望这个文件存在于文件系统中，请确保删除它。

还有其他方法可以运行后台脚本，比如通过接入系统和服务管理器，如 *systemd*。这些管理器提供了额外的功能，比如监控进程是否在运行，如果进程停止则重新启动，并捕获故障。如果你有这样的使用场景，我们建议你阅读有关 systemd 的更多资料，地址是 *[`man7.org/linux/man-pages/man1/init.1.html`](https://man7.org/linux/man-pages/man1/init.1.html)*。

## 渗透测试者的 Bash 自定义

作为渗透测试员，我们通常会遵循所有道德黑客参与的标准工作流程，无论是咨询工作、漏洞赏金狩猎，还是红队演练。我们可以通过一些 bash 技巧和窍门来优化这些工作。

### 将脚本放入可搜索路径

Bash 在由 PATH 环境变量定义的目录中搜索程序。像 ls 这样的命令始终可用，因为系统和用户的二进制文件位于 PATH 中的一些目录下。

要查看你的 PATH，运行以下命令：

```
$ **echo $PATH**

/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin 
```

输出可能会有所不同，具体取决于你的操作系统。

当你编写 bash 脚本时，可以将其放入如*/usr/local/bin* 这样的目录中，正如你所看到的，它是 PATH 的一部分。如果你不这样做，还有其他几种选择：

+   使用完整路径直接调用脚本。

+   将目录切换到脚本所在的目录，然后从那里执行它。

+   使用别名（在下一部分展示）。

+   将路径添加到 PATH 环境变量中。

将脚本放入可搜索路径的好处是，你可以仅通过命令名来调用它。你不需要提供完整路径，或者让终端位于同一目录中。

### 缩短命令与别名

当你经常使用一个较长的 Linux 命令时，可以使用*别名*将命令映射为一个较短的自定义名称，这样在需要运行时就能节省时间。

例如，假设你经常使用 Nmap（在第四章中讨论）带有特殊参数来扫描给定 IP 地址上的所有 65,535 个端口：

```
nmap -vv -T4 -p- -sV --max-retries 5 localhost
```

这个命令相当难记。使用别名后，我们可以使它在命令行或脚本中更容易访问。在这里，我们将命令分配给别名 quicknmap：

```
$ **alias quicknmap="nmap -vv -T4 -p- -sV --max-retries 5 localhost"**
```

现在我们可以通过使用别名的名称来运行别名命令：

```
$ **quicknmap**
Starting Nmap (https://nmap.org) at 02-21 22:32 EST
`--snip--`
PORT    STATE SERVICE
631/tcp open  ipp 
```

你甚至可以为自己的脚本分配别名：

```
$ **alias helloworld="bash ~/scripts/helloworld.sh"**
```

别名不是永久性的，但它们可以是永久的。在下一部分，你将学习如何使用 bash 配置文件使更改在 shell 中永久生效。

### 自定义 ~/.bashrc 配置文件

我们可以使用*~/.bashrc* 文件加载函数、变量和几乎任何其他自定义 bash 代码到新的 bash 会话中。例如，我们可以创建包含我们经常需要访问的信息的变量，比如我们正在测试的易受攻击主机的 IP 地址。

例如，我们可以将以下内容附加到*~/.bashrc* 文件的末尾。这些行定义了几个自定义变量，并保存了我们别名的 Nmap 命令：

```
VULN_HOST=1.0.0.22
VULN_ROUTER=10.0.0.254

alias quicknmap="nmap -vv -T4 -p- -sV --max-retries 5 localhost" 
```

下次打开终端时，你就可以访问这些值。通过使用 source 命令重新导入*~/.bashrc* 文件，可以立即使这些新值生效：

```
$ **source ~/.bashrc**

$ **echo ${VULN_HOST}**
10.0.0.22

$ **echo ${VULN_ROUTER}**
10.0.0.254 
```

现在，即使你关闭终端并启动一个新会话，你也可以使用这些变量。

### 导入自定义脚本

另一种引入更改到你的 bash 会话的方法是创建一个专门的脚本，包含渗透测试相关的自定义设置，然后让 *~/.bashrc* 文件通过 source 命令加载它。为此，创建一个 *~/.pentest.sh* 文件，包含你的新逻辑，然后对 *~/.bashrc* 文件做一次性修改，加载 *pentest.sh* 文件：

```
source ~/.pentest.sh
```

请注意，你还可以通过使用 `.`（点）命令来 source 一个 bash 文件：

```
. ~/.pentest.sh
```

这个命令提供了一个替代 source 的方法。

### 捕捉终端会话活动

渗透测试通常涉及同时打开数十个终端，每个终端运行多个工具，产生大量输出。当我们发现有用信息时，可能需要将一些输出作为证据保留下来。为了避免丢失重要信息，我们可以使用一些巧妙的 bash 技巧。

脚本命令允许我们捕捉终端会话活动。一个方法是加载一个小的 bash 脚本，使用 script 将每个会话保存到文件中，以便后续检查。这个脚本可能如下所示：清单 2-27。

```
#!/bin/bash

FILENAME=$(date +%m_%d_%Y_%H:%M:%S).log

if [[! -d ~/sessions]]; then
  mkdir ~/sessions
fi

# Starting a script session
if [[-z $SCRIPT]]; then
  export SCRIPT="/home/kali/sessions/${FILENAME}"
  script -q -f "${SCRIPT}"
fi 
```

清单 2-27：将终端活动保存到文件

如前所述，通过让 *~/.bashrc* 加载这个脚本，将会创建 *~/sessions* 目录，其中包含每个终端会话的捕获文件。录制将在你输入 `exit` 或关闭终端窗口时停止。

练习 2：Ping 一个域名

在这个练习中，你将编写一个 bash 脚本，接受两个参数：一个名称（例如，*mysite*）和一个目标域名（例如，*nostarch.com*）。该脚本应能执行以下操作：

1. 如果缺少参数，抛出错误并使用正确的退出代码退出。

2. 对域名进行 ping 测试，并返回 ping 是否成功的指示。要了解 ping 命令，可以运行 `man ping`。

3. 将结果写入包含以下信息的 CSV 文件：

a. 提供给脚本的名称

b. 提供给脚本的目标域名

c. ping 结果（成功或失败）

d. 当前的日期和时间

与 bash 中的大多数任务一样，有多种方法可以实现此目标。你可以在本书的 GitHub 仓库中找到这个练习的示例解决方案，*exercise_solution.sh*。

## 总结

在本章中，你学习了如何通过使用条件、循环和函数来执行流程控制；如何通过使用作业来控制脚本；以及如何搜索和解析文本。我们还强调了构建更有效渗透测试工作流的 bash 技巧和窍门。
