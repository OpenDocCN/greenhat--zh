## 第十二章：**OS X 脚本**

![image](img/common4.jpg)

Unix 及类 Unix 操作系统世界中的一个重要变化是完全重写的 OS X 系统的发布，这个系统建立在一个可靠的 Unix 内核上，名为 Darwin。Darwin 是一个基于 BSD Unix 的开源 Unix 系统。如果你了解 Unix，当你第一次在 OS X 中打开 Terminal 应用程序时，你无疑会欣喜若狂。最新一代的 Mac 电脑包括了你需要的一切，从开发工具到标准的 Unix 工具，配备了一个美丽的图形界面，非常适合那些还未准备好使用这些强大功能的用户。

然而，OS X 和 Linux/Unix 之间存在显著的差异，因此学习一些可以帮助你日常操作的 OS X 技巧是很有用的。例如，OS X 有一个有趣的命令行应用程序叫做`open`，它允许你从命令行启动图形应用程序。但`open`并不十分灵活。如果你想打开微软 Excel，输入`open excel`是行不通的，因为`open`比较挑剔，它期待你输入`open -a "Microsoft Excel"`。在本章后面，我们将编写一个包装脚本来绕过这个挑剔的行为。

**修复 OS X 行尾问题**

这里有另一种偶尔会遇到的情况，通过小小的调整可以变得更简单。如果你在命令行中处理为 Mac 的图形界面创建的文件，你会发现这些文件中的行尾字符与命令行需要的字符不一样。从技术术语来说，OS X 系统使用回车符（`\r`表示法）作为行尾，而 Unix 系统则使用换行符（`\n`）。所以，Mac 文件在终端中显示时不会有适当的换行。

有一个文件正遭遇这个问题吗？如果你尝试使用`cat`命令输出文件内容，你会看到下面的结果。

```
$ cat mac-format-file.txt
$
```

但你知道这个文件并不是空的。要查看其中有内容，使用`cat`命令的`-v`标志，它会使所有隐藏的控制字符可见。现在你会看到如下内容：

```
$ cat -v mac-format-file.txt
The rain in Spain^Mfalls mainly on^Mthe plain.^MNo kidding. It does.^M $
```

显然出了点问题！幸运的是，使用`tr`命令将回车符替换为正确的换行符非常简单。

```
$ tr '\r' '\n' < mac-format-file.txt > unix-format-file.txt
```

一旦将这个应用到示例文件，事情就变得更加明了。

```
$ tr '\r' '\n' < mac-format-file.txt
The rain in Spain
falls mainly on
the plain.
No kidding. It does.
```

如果你在像 Microsoft Word 这样的 Mac 应用程序中打开一个 Unix 文件，并且它看起来非常混乱，你也可以将行尾字符转换到另一个方向——转向 Aqua 应用程序。

```
$ tr '\n' '\r' < unixfile.txt > macfile.txt
```

嗯，这只是你在 OS X 中会看到的一些小差异之一。我们必须处理这些小怪癖，但也可以利用 OS X 的一些更好特性。

让我们开始吧，好吗？

### #79 自动化屏幕截图

如果你使用 Mac 电脑已经有一段时间了，你应该知道它内置了屏幕截图功能，通过按下![image](img/common2.jpg)-SHIFT-3 就能激活。你还可以使用 OS X 中的实用工具`Preview`或`Grab`，它们分别位于“应用程序”和“实用工具”文件夹中，也有很多出色的第三方工具可以选择。

但是你知道吗，其实有一个命令行的替代方案？这个超级有用的程序` screencapture `可以截取当前屏幕的截图，并将其保存到剪贴板或者指定的文件中（JPEG 或 TIFF 格式）。输入没有定义参数的命令，你将看到它的基本操作，如下所示：

```
$ screencapture -h
screencapture: illegal option -- h
usage: screencapture [-icMPmwsWxSCUtoa] [files]
  -c         force screen capture to go to the clipboard
  -C         capture the cursor as well as the screen. only in non-interactive
modes
  -d         display errors to the user graphically
  -i         capture screen interactively, by selection or window
               control key - causes screen shot to go to clipboard
               space key   - toggle between mouse selection and
                             window selection modes
               escape key  - cancels interactive screen shot
  -m         only capture the main monitor, undefined if -i is set
  -M         screen capture output will go to a new Mail message
  -o         in window capture mode, do not capture the shadow of the window
  -P         screen capture output will open in Preview
  -s         only allow mouse selection mode
  -S         in window capture mode, capture the screen not the window
  -t<format> image format to create, default is png (other options include
pdf, jpg, tiff and other formats)
  -T<seconds> Take the picture after a delay of <seconds>, default is 5
  -w         only allow window selection mode
  -W         start interaction in window selection mode
  -x         do not play sounds
  -a         do not include windows attached to selected windows
  -r         do not add dpi meta data to image
  -l<windowid> capture this windowsid
  -R<x,y,w,h> capture screen rect
  files   where to save the screen capture, 1 file per screen
```

这是一个非常需要包装脚本的应用程序。例如，要在 30 秒后截取屏幕截图，你可以使用以下命令：

```
$ sleep 30; screencapture capture.tiff
```

但是，让我们做些更有趣的事情，好吗？

#### *代码*

列表 11-1 显示了我们如何自动化` screencapture `工具，以便它能更加隐秘地截取屏幕截图。

```
   #!/bin/bash
   # screencapture2--Use the OS X screencapture command to capture a sequence of
   #   screenshots of the main window, in stealth mode. Handy if you're in a
   #   questionable computing environment!

   capture="$(which screencapture) -x -m -C"
➊ freq=60         # Every 60 seconds
   maxshots=30     # Max screen captures
   animate=0       # Create animated gif? No.

   while getopts "af:m" opt; do
     case $opt in
      a ) animate=1;                  ;;
      f ) freq=$OPTARG;               ;;
      m ) maxshots=$OPTARG;           ;;  # Quit after specified num of pics
      ? ) echo "Usage: $0 [-a] [-f frequency] [-m maxcaps]" >&2
          exit 1
     esac
   done

   counter=0

   while [ $counter -lt $maxshots ] ; do
     $capture capture${counter}.jpg   # Counter keeps incrementing.
     counter=$(( counter + 1 ))
     sleep $freq   # freq is therefore the number of seconds between pics.
   done

   # Now, optionally, compress all the individual images into an animated GIF.

   if [ $animate -eq 1 ] ; then
➋   convert -delay 100 -loop 0 -resize "33%" capture* animated-captures.gif
   fi

   # No exit status to stay stealthy
   exit 0
```

*列表 11-1：`*screencapture2*`包装脚本*

#### *工作原理*

这将在每个`$freq`秒 ➊ 截取一张截图，直到达到`$maxshots`次截图（默认为每 60 秒截取一次，总共 30 次）。输出是一系列的 JPEG 文件，按顺序编号，从 0 开始。这对训练目的非常有用，或者如果你怀疑有人在你午休时使用了你的电脑：设置这个，然后你可以在没有人察觉的情况下回顾发生的事情。

脚本的最后部分很有趣：它可选择通过使用 ImageMagick 的`convert`工具 ➋ 生成一个原始图像大小的三分之一的动画 GIF。这是一个非常方便的方式，可以一次性查看所有图像。在第十四章中，我们会更多地使用 ImageMagick！你可能在 OS X 系统上没有默认安装这个命令，但是通过使用像`brew`这样的包管理工具，你可以通过一个命令安装它（`brew install imagemagick`）。

#### *运行脚本*

因为这段代码是设计用来在后台隐秘运行的，所以基本的调用方式很简单：

```
$ screencapture2 &
$
```

就是这么简单。作为示例，要指定截取多少次截图（30 次）以及何时截取（每 5 秒一次），你可以像这样启动`screencapture2`脚本：

```
$ screencapture2 -f 5 -m 30 &
$
```

#### *结果*

运行脚本不会有任何输出，但会出现新文件，如列表 11-2 所示。（如果你指定了`-a`动画标志，你将会看到额外的结果。）

```
$ ls -s *gif *jpg
 4448 animated-captures.gif      4216 capture2.jpg      25728 capture5.jpg
 4304 capture0.jpg               4680 capture3.jpg      4456 capture6.jpg
 4296 capture1.jpg               4680 capture4.jpg
```

*列表 11-2：通过`*screencapture2*`捕捉的屏幕图像，记录了一个时间段内的截图*

#### *破解脚本*

对于一个长期的屏幕监视工具，你需要找到一种方法来检查屏幕何时真正发生变化，这样就不会用无趣的屏幕截图浪费硬盘空间。有一些第三方解决方案可以让`screencapture`运行更长时间，保存屏幕实际变化的历史，而不是保存成百上千份相同的、未改变的屏幕截图。（请注意，如果你的屏幕上有时钟显示，每一张屏幕截图都会稍微不同，这会让你更难避免这个问题！）

借助这个功能，你可以将“monitor ON”和“monitor OFF”作为一个包装器，启动捕捉序列并分析图像是否与第一次捕捉的不同。但是，如果你使用这个脚本的 GIF 来制作在线培训教程，你可能会使用更精细的控制来设置捕捉的时长，并将这一时长作为命令行参数。

### #80 动态设置终端标题

列出 11-3 是一个有趣的小脚本，适用于喜欢在终端应用程序中工作的 OS X 用户。你不再需要使用**终端** ![image](img/common3.jpg) **偏好设置** ![image](img/common3.jpg) **配置文件** ![image](img/common3.jpg) **窗口**对话框来设置或更改窗口标题，而是可以使用此脚本随时更改它。在这个例子中，我们将通过将当前工作目录包含在内，让终端窗口的标题变得更加实用。

#### *代码*

```
   #!/bin/bash
   # titleterm--Tells the OS X Terminal application to change its title
   #   to the value specified as an argument to this succinct script

   if [ $# -eq 0 ]; then
     echo "Usage: $0 title" >&2
     exit 1
   else
➊   echo -e "\033]0;$@\007"
   fi

   exit 0
```

*列出 11-3：* `*titleterm*` *脚本*

#### *它是如何工作的*

终端应用程序有多种它能识别的秘密转义码，而`titleterm`脚本会发送一串`ESC ] 0; title BEL` ➊，这会将标题更改为指定的值。

#### *运行脚本*

要更改终端窗口的标题，只需将你想要的标题作为参数输入`titleterm`即可。

#### *结果*

命令没有明显的输出，正如列出 11-4 所示。

```
$ titleterm $(pwd)
$
```

*列出 11-4：运行* `*titleterm*` *脚本，将终端标题设置为当前目录的标题*

然而，它会立即将终端窗口的标题更改为当前工作目录。

#### *破解脚本*

只需在登录脚本（* .bash_profile * 或根据你使用的登录 shell 选择其他文件）中添加一个小的修改，就可以让终端窗口的标题自动显示当前的工作目录。例如，要使这段代码显示你当前的工作目录，你可以在`tcsh`中使用以下代码：

```
alias precmd 'titleterm "$PWD"'                      [tcsh]
```

或者在`bash`中使用这个：

```
export PROMPT_COMMAND="titleterm \"\$PWD\""          [bash]
```

只需将上述命令之一放入登录脚本中，从下次打开终端窗口开始，你会发现每次进入新目录时，窗口标题都会发生变化。真是非常有用。

### #81 生成 iTunes 库的汇总列表

如果你使用 iTunes 已有一段时间，肯定会有一个庞大的音乐、有声书、电影和电视节目列表。不幸的是，尽管 iTunes 功能强大，但并没有一个简单的方法以简洁易读的格式导出你的音乐列表。幸运的是，编写一个提供此功能的脚本并不困难，列表 11-5 就展示了这个脚本。这个脚本依赖于 iTunes 的“与其他应用程序共享 iTunes XML”功能，因此在运行此脚本之前，确保在 iTunes 偏好设置中启用了此功能。

#### *代码*

```
   #!/bin/bash
   # ituneslist--Lists your iTunes library in a succinct and attractive
   #   manner, suitable for sharing with others, or for synchronizing
   #   (with diff) iTunes libraries on different computers and laptops

   itunehome="$HOME/Music/iTunes"
   ituneconfig="$itunehome/iTunes Music Library.xml"

➊ musiclib="/$(grep '>Music Folder<' "$ituneconfig" | cut -d/ -f5- | \
     cut -d\< -f1 | sed 's/%20/ /g')"

   echo "Your library is at $musiclib"

   if [ ! -d "$musiclib" ] ; then
     echo "$0: Confused: Music library $musiclib isn't a directory?" >&2
     exit 1
   fi

   exec find "$musiclib" -type d -mindepth 2 -maxdepth 2 \! -name '.*' -print \
     | sed "s|$musiclib/||"
```

*列表 11-5：* `*ituneslist*` *脚本*

#### *工作原理*

像许多现代计算机应用程序一样，iTunes 希望其音乐库位于一个标准位置——在这个例子中是 *~/Music/iTunes/iTunes Media/*——但也允许你将其移到其他位置。脚本需要能够确定不同的位置，这可以通过从 iTunes 偏好设置文件中提取 `Music Folder` 字段值来完成。这正是 ➊ 处管道命令的作用。

偏好设置文件（`$ituneconfig`）是一个 XML 数据文件，因此需要一些切割操作来确定准确的 `Music Folder` 字段值。以下是 Dave 的 iTunes 配置文件中 `iTunes Media` 值的样子：

```
file://localhost/Users/taylor/Music/iTunes/iTunes %20Media/
```

`iTunes Media` 值实际上是以完全限定的 URL 存储的，颇为有趣，因此我们需要去掉 *file://localhost/* 前缀。这是第一个 `cut` 命令的工作。最后，由于许多 OS X 目录中包含空格，且 `Music Folder` 字段以 URL 格式保存，该字段中的所有空格都被映射为 `%20` 序列，必须通过 `sed` 命令将其还原为空格，然后才能继续操作。

确定了 `Music Folder` 名称后，现在可以很容易地在两台 Mac 系统上生成音乐列表，然后使用 `diff` 命令进行比较，这使得查看哪些专辑是某一系统独有的变得轻松，或许可以进行同步。

#### *运行脚本*

这个脚本没有命令参数或标志。

#### *结果*

如果你有一个庞大的音乐收藏，脚本的输出可能会非常大。列表 11-6 显示了 Dave 音乐收藏输出的前 15 行。

```
$ ituneslist | head -15
Your library is at /Users/taylor/Music/iTunes/iTunes Media/
Audiobooks/Andy Weir
Audiobooks/Barbara W. Tuchman
Audiobooks/Bill Bryson
Audiobooks/Douglas Preston
Audiobooks/Marc Seifer
Audiobooks/Paul McGann
Audiobooks/Robert Louis Stevenson
iPod Games/Klondike
Movies/47 Ronin (2013)
Movies/Mad Max (1979)
Movies/Star Trek Into Darkness (2013)
Movies/The Avengers (2012)
Movies/The Expendables 2 (2012)
Movies/The Hobbit The Desolation of Smaug (2013)
```

*列表 11-6：运行* `*ituneslist*` *脚本以打印 iTunes 收藏中的顶级项目*

#### *修改脚本*

好吧，这不完全是关于修改脚本本身的，但由于 iTunes 库目录是作为完全限定的 URL 存储的，尝试将 iTunes 目录设为可以通过 Web 访问的目录，并将该目录的 URL 作为 XML 文件中的 `Music Folder` 值，应该会很有趣......

### #82 修复 open 命令

OS X 的一项有趣创新是增加了 `open` 命令，它可以让你轻松启动几乎任何类型文件的相应应用程序，无论是图形图像、PDF 文档还是 Excel 表格。`open` 命令的问题在于它有些古怪。如果你想让它启动一个指定的应用程序，你必须包含 `-a` 标志。如果你没有指定准确的应用程序名称，它会报错并失败。这正是像 清单 11-7 中的封装脚本可以解决的问题。

#### *代码*

```
   #!/bin/bash
   # open2--A smart wrapper for the cool OS X 'open' command
   #   to make it even more useful. By default, 'open' launches the
   #   appropriate application for a specified file or directory
   #   based on the Aqua bindings, and it has a limited ability to
   #   launch applications if they're in the /Applications dir.

   #   First, whatever argument we're given, try it directly.

➊ if ! open "$@" >/dev/null 2>&1 ; then
     if ! open -a "$@" >/dev/null 2>&1 ; then

       # More than one arg? Don't know how to deal with it--quit.
       if [ $# -gt 1 ] ; then
         echo "open: More than one program not supported" >&2
         exit 1
       else
➋         case $(echo $1 | tr '[:upper:]' '[:lower:]') in
           activ*|cpu   ) app="Activity Monitor"           ;;
           addr*        ) app="Address Book"               ;;
           chat         ) app="Messages"                   ;;
           dvd          ) app="DVD Player"                 ;;
           excel        ) app="Microsoft Excel"            ;;
           info*        ) app="System Information"         ;;
           prefs        ) app="System Preferences"         ;;
           qt|quicktime ) app="QuickTime Player"           ;;
           word         ) app="Microsoft Word"             ;;
           *            ) echo "open: Don't know what to do with $1" >&2
               exit 1
         esac
         echo "You asked for $1 but I think you mean $app." >&2
         open -a "$app"
       fi
     fi
   fi

   exit 0
```

*清单 11-7：* `*open2*` *脚本*

#### *工作原理*

这个脚本围绕零返回码和非零返回码展开，其中 `open` 程序在成功时返回零代码，在失败时返回非零代码 ➊。

如果提供的参数不是文件名，第一个条件判断会失败，脚本会通过添加 `a` 来测试提供的参数是否是有效的应用程序名称。如果第二个条件判断失败，脚本会使用 `case` 语句 ➋ 来检查人们常用来指代流行应用程序的常见昵称。

它甚至会在匹配到昵称时提供友好的提示信息，然后再启动指定的应用程序。

```
$ open2 excel
You asked for excel but I think you mean Microsoft Excel.
```

#### *运行脚本*

`open2` 脚本要求在命令行中指定一个或多个文件名或应用程序名称。

#### *结果*

如果没有这个封装程序，尝试打开 Microsoft Word 应用程序会失败。

```
$ open "Microsoft Word"
The file /Users/taylor/Desktop//Microsoft Word does not exist.
```

尽管出现了一条相当吓人的错误信息，但那仅仅是因为用户没有提供 `-a` 标志。使用 `open2` 脚本相同的调用则表明，不再需要记住 `-a` 标志：

```
$ open2 "Microsoft Word"
$
```

没有输出是好事：应用程序已启动并准备就绪。此外，常见 OS X 应用程序的昵称系列意味着，虽然 `open -a word` 绝对无法使用，但 `open2 word` 则能正常工作。

#### *修改脚本*

如果昵称列表根据你的具体需求或用户社区的需求进行了定制，这个脚本会变得更加有用。那应该很容易做到！
