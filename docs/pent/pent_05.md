## 第三章. 编程

在本章中，我们将查看一些计算机编程的基本示例。我们将编写程序来自动化执行各种有用的任务，使用多种编程语言。尽管本书的大部分内容都使用了预构建的软件，但能够编写自己的程序仍然很有用。

## Bash 脚本编写

本节我们将介绍如何使用 Bash 脚本一次性运行多个命令。*Bash 脚本*，或称 *Shell 脚本*，是包含多个终端命令的文件，这些命令会依次执行。我们在终端中可以运行的任何命令，都可以在脚本中执行。

### Ping

我们将把第一个脚本命名为 *pingscript.sh*。当它运行时，这个脚本会对我们本地网络进行 *ping 扫描*，向远程系统发送互联网控制消息协议（ICMP）消息，以查看它们是否响应。

我们将使用 ping 工具来确定网络上哪些主机是可达的。（尽管有些主机可能不会响应 ping 请求，并且尽管它们无法“ping 通”，它们仍然可能是正常运行的，但 ping 扫描仍然是一个很好的起点。）默认情况下，我们提供 IP 地址或主机名进行 ping 测试。例如，要 ping 我们的 Windows XP 目标，请在 示例 3-1 中输入粗体代码。

示例 3-1. 远程主机 Ping 测试

```
root@kali:~/# ping 192.168.20.10
PING 192.168.20.10 (192.168.20.10) 56(84) bytes of data.
64 bytes from 192.168.20.10: icmp_req=1 ttl=64 time=0.090 ms
64 bytes from 192.168.20.10: icmp_req=2 ttl=64 time=0.029 ms
64 bytes from 192.168.20.10: icmp_req=3 ttl=64 time=0.038 ms
64 bytes from 192.168.20.10: icmp_req=4 ttl=64 time=0.050 ms
**^C**
--- 192.168.20.10 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 2999 ms
rtt min/avg/max/mdev = 0.029/0.051/0.090/0.024 ms
```

从 ping 输出中我们可以看出，Windows XP 目标已启动并响应了 ping 探测，因为我们收到了对 ICMP 请求的回复。（ping 的问题在于，除非你用 ctrl-C 停止它，否则它会一直运行下去。）

### 一个简单的 Bash 脚本

让我们开始编写一个简单的 Bash 脚本来 ping 网络上的主机。一个好的起点是添加一些帮助信息，告诉用户如何正确运行你的脚本。

```
#**!/bin/bash**
**echo "Usage: ./pingscript.sh [network]"**
**echo "example: ./pingscript.sh 192.168.20"**
```

这个脚本的第一行告诉终端使用 Bash 解释器。接下来的两行以 *echo* 开头，简单地告诉用户我们的 ping 脚本将接受一个命令行参数（网络），告诉脚本要进行 ping 扫描的网络（例如，192.168.20）。`echo` 命令会将引号中的文本打印出来。

### 注意

这个脚本意味着我们正在处理一个 C 类网络，其中 IP 地址的前三个八位字节组成网络部分。

创建脚本后，使用 `chmod` 命令将其设为可执行文件，以便我们可以运行它。

```
root@kali:~/# chmod 744 pingscript.sh
```

### 运行我们的脚本

以前，当输入 Linux 命令时，我们会在提示符下键入命令名。内置的 Linux 命令以及添加到 Kali Linux 中的渗透测试工具的文件系统位置是我们 `PATH` 环境变量的一部分。`PATH` 变量告诉 Linux 在哪些目录中查找可执行文件。要查看 `PATH` 中包含了哪些目录，请输入 **`echo $PATH`**。

```
root@kali:~/# echo $PATH
/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
```

注意输出中没有列出 */root* 目录。这意味着我们不能简单地输入 `pingscript.sh` 来运行我们的 Bash 脚本。相反，我们需要输入 **`./pingscript.sh`** 来告诉终端从当前目录运行脚本。如下所示，脚本会打印出使用信息。

```
root@kali:~/# ./pingscript.sh
Usage: ./pingscript.sh [network]
example: ./pingscript.sh 192.168.20
```

### 使用`if`语句增加功能

现在，让我们通过添加一个`if`语句来增加一些功能，如示例 3-2 所示。

示例 3-2. 添加`if`语句

```
#!/bin/bash
if [ "$1" == "" ] ❶
then ❷
echo "Usage: ./pingscript.sh [network]"
echo "example: ./pingscript.sh 192.168.20"
fi ❸
```

通常，脚本只有在用户使用不当时才需要打印使用信息。在这种情况下，用户需要提供网络扫描的命令行参数。如果用户没有这样做，我们希望通过打印使用信息来告知用户如何正确运行脚本。

为了实现这一点，我们可以使用`if`语句来判断条件是否满足。通过使用`if`语句，我们可以在特定条件下让脚本仅在某些情况下输出使用信息——例如，当用户未提供命令行参数时。

`if`语句在许多编程语言中都有使用，尽管语法在不同语言中有所不同。在 Bash 脚本中，`if`语句的使用方式如下：`if [`*`condition`*`]`，其中`*condition*`是必须满足的条件。

在我们的脚本中，我们首先检查第一个命令行参数是否为空 ❶。符号`$1`表示在 Bash 脚本中的第一个命令行参数，双等号（`==`）用于检查是否相等。`if`语句后面跟着一个`then`语句 ❷。`then`语句和`fi`（`if`的倒序）之间的任何命令 ❸ 只有在条件语句为真时才会执行——在这种情况下，当脚本的第一个命令行参数为空时。

当我们在没有命令行参数的情况下运行新的脚本时，`if`语句的结果为真，因为第一个命令行参数确实为空，如下所示。

```
root@kali:~/# ./pingscript.sh
Usage: ./pingscript.sh [network]
example: ./pingscript.sh 192.168.20
```

正如预期的那样，我们看到了使用信息被回显到屏幕上。

### `for`循环

如果我们再次使用命令行参数运行脚本，什么也不会发生。现在，让我们添加一些功能，当用户使用正确的参数运行脚本时会触发这些功能，如示例 3-3 所示。

示例 3-3. 添加`for`循环

```
#!/bin/bash
if [ "$1" == "" ]
then
echo "Usage: ./pingscript.sh [network]"
echo "example: ./pingscript.sh 192.168.20"
**else** ❶
**for x in `seq 1 254`; do** ❷
**ping -c 1 $1.$x**
**done** ❸
fi
```

在`then`语句后，我们使用`else`语句 ❶ 来指示脚本在`if`语句为假时执行代码——在这种情况下，如果用户提供了命令行参数。因为我们希望这个脚本能够对本地网络上的所有主机进行 ping 操作，所以我们需要遍历 1 到 254 之间的数字（IP 版本 4 地址最后一个八位字节的可能性），并对这些可能性执行`ping`命令。

一种理想的方式是使用`for`循环 ❷ 来遍历顺序的可能性。我们的`for`循环`for x in \`seq 1 254\`; do`告诉脚本对从 1 到 254 的每个数字执行后续代码。这将允许我们运行一组指令 254 次，而不是为每个实例编写代码。我们用`done` ❸ 指令表示`for`循环的结束。

在 `for` 循环内部，我们希望对网络中的每个 IP 地址进行 ping 操作。通过查看 ping 的 man 页面，我们发现 `-c` 选项可以让我们限制每次 ping 操作的次数。我们将 `-c` 设置为 `1`，这样每个主机只会被 ping 一次。

为了指定要 ping 的主机，我们需要将第一个命令行参数（表示前三个八位组）与 `for` 循环的当前迭代进行拼接。使用的完整命令是 `ping -c 1 $1.$x`。回想一下，`$1` 表示第一个命令行参数，`$x` 是 `for` 循环的当前迭代。我们的 `for` 循环第一次运行时，它将 ping 192.168.20.1，然后是 192.168.20.2，一直到 192.168.20.254。迭代到 254 后，`for` 循环结束。

当我们使用 IP 地址前三个八位组作为命令行参数运行脚本时，脚本将对网络中的每个 IP 地址进行 ping 操作，如 示例 3-4 中所示。

示例 3-4. 运行 ping 扫描脚本

```
root@kali:~/# ./pingscript.sh 192.168.20
PING 192.168.20.1 (192.168.20.1) 56(84) bytes of data.
64 bytes from 192.168.20.1: icmp_req=1 ttl=255 time=8.31 ms ❶

--- 192.168.20.1 ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 8.317/8.317/8.317/0.000 ms
PING 192.168.20.2(192.168.20.2) 56(84) bytes of data.
64 bytes from 192.168.20.2: icmp_req=1 ttl=128 time=166 ms

--- 192.168.20.2 ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 166.869/166.869/166.869/0.000 ms
PING 192.168.20.3 (192.168.20.3) 56(84) bytes of data.
From 192.168.20.13 icmp_seq=1 Destination Host Unreachable ❷

--- 192.168.20.3 ping statistics ---
1 packets transmitted, 0 received, +1 errors, 100% packet loss, time 0ms
--*snip*--
```

你的结果会根据你本地网络中的系统有所不同。根据这个输出，我可以看出在我的网络中，主机 192.168.20.1 正在运行，并且我收到了一个 ICMP 响应 ❶。另一方面，主机 192.168.20.3 未启动，因此我收到了主机不可达的通知 ❷。

### 精简结果

所有这些打印到屏幕上的信息看起来并不太好，任何使用我们脚本的人都需要筛选出大量的信息才能确定网络中哪些主机是正常的。让我们添加一些额外的功能来精简结果。

在上一章中我们介绍了 `grep`，它用于搜索和匹配特定的模式。让我们使用 `grep` 来过滤脚本的输出，如 示例 3-5 中所示。

示例 3-5. 使用 `grep` 过滤结果

```
#!/bin/bash
if [ "$1" == "" ]
then
echo "Usage: ./pingscript.sh [network]"
echo "example: ./pingscript.sh 192.168.20"
else
for x in `seq 1 254`; do
ping -c 1 $1.$x | grep "64 bytes" ❶
done
fi
```

在这里，我们查找所有包含字符串 `64 bytes` ❶ 的实例，这个字符串出现在 ping 主机时收到 ICMP 响应的情况下。如果我们使用这个修改后的脚本运行，我们会看到只有包含 `64 bytes` 文本的行被打印到屏幕上，如下所示。

```
root@kali:~/# ./pingscript.sh 192.168.20
64 bytes from 192.168.20.1: icmp_req=1 ttl=255 time=4.86 ms
64 bytes from 192.168.20.2: icmp_req=1 ttl=128 time=68.4 ms
64 bytes from 192.168.20.8: icmp_req=1 ttl=64 time=43.1 ms
--*snip*--
```

我们只获得活动主机的指示；那些没有回应的主机不会打印到屏幕上。

但是我们可以让这个脚本变得更易于使用。我们的 ping 扫描的目的是获取活动主机的列表。通过使用 第二章 中讨论的 `cut` 命令，我们可以仅打印出活动主机的 IP 地址，如 示例 3-6 中所示。

示例 3-6. 使用 `cut` 进一步过滤结果

```
#!/bin/bash
if [ "$1" == "" ]
then
echo "Usage: ./pingscript.sh [network]"
echo "example: ./pingscript.sh 192.168.20"
else
for x in `seq 1 254`; do
ping -c 1 $1.$x | grep "64 bytes" | cut -d" " -f4 ❶
done
fi
```

我们可以使用空格作为分隔符，并获取第四个字段，也就是我们的 IP 地址，如 ❶ 所示。

现在我们再次运行脚本，如下所示。

```
root@kali:~/mydirectory# ./pingscript.sh 192.168.20
192.168.20.1:
192.168.20.2:
192.168.20.8:
--*snip*--
```

不幸的是，我们看到每一行的末尾都有一个冒号。对于用户来说，结果应该已经足够清晰，但如果我们想将这些结果作为输入传递给其他程序，我们需要删除末尾的冒号。在这种情况下，`sed` 就是解决方案。

将删除每行末尾字符的 `sed` 命令是 `sed 's/.$//'`，如 示例 3-7 所示。

示例 3-7. 使用 `sed` 删除末尾的冒号

```
#!/bin/bash
if [ "$1" == "" ]
then
echo "Usage: ./pingscript.sh [network]"
echo "example: ./pingscript.sh 192.168.20"
else
for x in `seq 1 254`; do
ping -c 1 $1.$x | grep "64 bytes" | cut -d" " -f4 | **sed 's/.$//'**
done
fi
```

现在当我们运行脚本时，一切看起来都完美无缺，如下所示。

```
root@kali:~/# ./pingscript.sh 192.168.20
192.168.20.1
192.168.20.2
192.168.20.8
--*snip*--
```

### 注意

当然，如果我们希望将结果输出到文件中，而不是显示在屏幕上，我们可以使用 `>>` 操作符，这在 第二章 中有介绍，用来将每个存活的 IP 地址追加到文件中。尝试在 Linux 中自动化其他任务，以练习你的 Bash 脚本技能。

## Python 脚本

Linux 系统通常预装有其他脚本语言的解释器，如 Python 和 Perl。Kali Linux 中包含了这两种语言的解释器。在 第十六章 到 第十九章 中，我们将使用 Python 编写自己的漏洞利用代码。目前，让我们编写一个简单的 Python 脚本，并在 Kali Linux 中运行，以演示 Python 脚本的基础知识。

对于这个示例，我们将做一些类似于 第二章 中我们第一个 Netcat 示例的事情：我们将连接到系统上的一个端口，并检查该端口是否在监听。我们脚本的起点如下所示。

```
#!/usr/bin/python ❶
ip = raw_input("Enter the ip: ") ❷
port = input("Enter the port: ") ❸
```

在上一节中，我们脚本的第一行告诉终端使用 Bash 来解释脚本。在这里我们做了相同的事情，指向 Kali Linux 上安装的 Python 解释器，路径为 */usr/bin/python* ❶。

我们将首先提示用户输入数据，并将输入记录到变量中。变量将存储输入的数据，以便稍后在脚本中使用。为了从用户那里获取输入，我们可以使用 Python 函数 `raw_input` ❷。我们希望将端口保存为整数，因此在 ❸ 处，我们使用了一个类似的内建 Python 函数 `input`。

保存文件后，使用 `chmod` 命令使脚本可执行，然后运行脚本，如下所示。

```
root@kali:~/mydirectory# chmod 744 pythonscript.py
root@kali:~/mydirectory# ./pythonscript.py
Enter the ip: 192.168.20.10
Enter the port: 80
```

当你运行脚本时，系统会提示你输入 IP 地址和端口，这是预期的行为。

现在，我们将添加一些功能，允许我们使用用户的输入连接到选择的系统的指定端口，以检查端口是否开放（示例 3-8）。

示例 3-8. 添加端口扫描功能

```
#!/usr/bin/python
import socket ❶
ip = raw_input("Enter the ip: ")
port = input("Enter the port: ")
s = socket.socket(socket.AF_INET, socket.SOCK_STREAM) ❷
if s.connect_ex((ip, port)): ❸
        print "Port", port, "is closed" ❹
else: ❺
        print "Port", port, "is open"
```

为了在 Python 中执行网络任务，我们可以使用命令 `import socket` ❶ 来引入一个名为 *socket* 的库。socket 库负责设置网络套接字的繁重工作。

创建 TCP 网络套接字的语法是`socket.socket(socket.AF_INET, socket.SOCK_STREAM)`。我们在❷处将一个变量设置为该网络套接字。

### 连接到端口

在创建一个连接到远程端口的套接字时，Python 中的第一个可选函数是`connect`。然而，对于我们的目的，有一个更好的选择，即类似的`connect_ex`函数。根据 Python 文档，`connect_ex`与`connect`类似，区别在于如果连接失败，它会返回错误代码，而不是引发异常。如果连接成功，`connect_ex`将返回值`0`。由于我们希望知道函数是否能成功连接到端口，这个返回值似乎非常适合用于`if`语句。

### Python 中的 if 语句

在 Python 中构建`if`语句时，我们输入`if` *`condition`*`:`。在 Python 中，属于条件`or`循环的语句通过缩进来表示，而不是像 Bash 脚本那样使用结束标记。我们可以指示`if`语句评估 TCP 套接字连接到用户定义的 IP 地址和端口的返回值，命令为`if s.connect_ex((ip, port)):` ❸。如果连接成功，`connect_ex`将返回`0`，这会被`if`语句评估为假。如果连接失败，`connect_ex`将返回一个正整数，或为真。因此，如果我们的`if`语句评估为真，可以推测端口已关闭，并且我们可以通过 Python 的`print`命令在❹处将结果展示给用户。与 Bash 脚本示例一样，如果`connect_ex`在❺处返回`0`，我们可以使用`else`语句（Python 中的语法是`else:`）来通知用户测试的端口是开放的。

现在，运行更新后的脚本测试目标主机上是否运行着 TCP 端口 80，如下所示。

```
root@kali:~/# ./pythonscript.py
Enter the ip: 192.168.20.10
Enter the port: 80
Port 80 is open
```

根据我们的脚本，端口 80 是开放的。现在再次运行脚本，测试端口 81。

```
root@kali:~/# ./pythonscript.py
Enter the ip: 192.168.20.10
Enter the port: 81
Port 81 is closed
```

这次，脚本报告端口 81 已关闭。

### 注意

我们将在第五章中了解如何检查开放端口，稍后在学习漏洞开发时会回到 Python 脚本。Kali Linux 还支持 Perl 和 Ruby 语言的解释器。我们将在第十九章学习一点 Ruby。掌握多种语言总是有益的。如果你有挑战精神，可以尝试用 Perl 和 Ruby 重新创建这个脚本。

## 编写和编译 C 程序

让我们做一个简单的编程示例，这次使用 C 编程语言。与 Bash 和 Python 等脚本语言不同，C 代码必须先编译并转换成 CPU 能理解的机器语言，然后才能运行。

Kali Linux 包含 GNU 编译器集合（GCC），这将允许我们编译 C 代码并在系统上运行。让我们创建一个简单的 C 程序，向命令行参数问好，如 示例 3-9 所示。

示例 3-9. “Hello World” C 程序

```
#include <stdio.h> ❶
int main(int argc, char *argv[]) ❷
{
    if(argc < 2) ❸
    {
        printf("%s\n", "Pass your name as an argument"); ❹
        return 0; ❺
    }
    else
    {
                printf("Hello %s\n", argv[1]); ❻
                return 0;
    }
}
```

C 的语法与 Python 和 Bash 有点不同。因为我们的代码将被编译，我们不需要告诉终端使用哪个解释器来执行代码。首先，像我们的 Python 示例一样，我们导入一个 C 库。在这种情况下，我们将导入 *stdio*（标准输入输出的缩写）库，它将允许我们接受输入并将输出打印到终端。在 C 中，我们使用命令 `#include <stdio.h>` 来导入 *stdio* ❶。

每个 C 程序都有一个名为 `main` 的函数 ❷，它在程序启动时运行。我们的程序将接受一个命令行参数，因此我们将一个整数 `argc` 和一个字符数组 `argv` 传递给 `main`。`argc` 是参数计数，`argv` 是参数向量，包含传递给程序的任何命令行参数。这是 C 程序接受命令行参数的标准语法。（在 C 中，函数、循环等的开始和结束由大括号 `{}` 标记。）

首先，我们的程序检查是否提供了命令行参数。`argc` 整数是参数数组的长度；如果它小于二（即程序名称和命令行参数），则说明没有提供命令行参数。我们可以使用 `if` 语句来进行检查 ❸。

`if` 语法在 C 中也有些不同。和 Bash 脚本一样，如果没有提供命令行参数，我们可以提示用户查看使用信息 ❹。`printf` 函数允许我们将输出写入终端。同时请注意，C 语言中的语句以分号（`;`）结束。一旦程序执行完毕，我们使用 `return` 语句 ❺ 来结束 `main` 函数。如果提供了命令行参数，我们的 `else` 语句指示程序向命令行参数问好 ❻。（确保使用大括号来闭合所有循环和 `main` 函数。）

在运行程序之前，我们需要用 GCC 编译它，如下所示。将程序保存为 *cprogram.c*。

```
root@kali:~# gcc cprogram.c -o cprogram
```

使用 `-o` 选项来指定编译后的程序名称，并将 C 代码传递给 GCC。现在从当前目录运行该程序。如果程序未带任何参数运行，你应该会看到如下的使用信息。

```
root@kali:~# ./cprogram
Pass your name as an argument
```

如果我们传递给它一个参数，在这种情况下是我们的名字，程序将向我们问好。

```
root@kali:~# ./cprogram georgia
Hello georgia
```

### 注意

我们将在 第十六章 中查看另一个 C 编程示例，其中一些不规范的 C 编程导致了缓冲区溢出条件，我们将对此进行利用。

## 总结

在本章中，我们查看了三种不同语言中的简单程序。我们了解了基本构造，比如将信息保存在变量中以供后续使用。此外，我们还学习了如何使用条件语句，如`if`语句，以及迭代语句，如`for`循环，让程序根据提供的信息做出决策。尽管不同编程语言的语法各异，但思想是相同的。
