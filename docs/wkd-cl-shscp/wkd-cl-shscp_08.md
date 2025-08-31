## 第八章：**网页和互联网用户**

![image](img/common4.jpg)

Unix 真正闪光的一个领域就是互联网。无论你是想在桌子底下运行一个快速的服务器，还是仅仅想高效智能地浏览网页，当涉及到互联网交互时，几乎没有什么是你不能嵌入 shell 脚本中的。

互联网工具是可以脚本化的，即使你可能从未想过它们是这样的。例如，FTP，这个始终处于调试模式的程序，可以用一些非常有趣的方式来编写脚本，这在 脚本 #53 中有探讨，位于 第 174 页。Shell 脚本通常可以提高大多数命令行工具在与互联网相关的方面的性能和输出。

本书的第一版曾向读者保证，互联网脚本编写者工具箱中最好的工具是 `lynx`；现在我们推荐使用 `curl`。这两个工具都提供纯文本界面来访问网页，但 `lynx` 尝试提供类似浏览器的体验，而 `curl` 则专门为脚本设计，能够提取任何页面的原始 HTML 源代码，供你查看。

例如，以下显示了通过 `curl` 获取的 *Dave on Film* 首页源代码的前七行：

```
$ curl -s http://www.daveonfilm.com/ | head -7
<!DOCTYPE html>
<html lang="en-US">
<head>
<meta charset="UTF-8" />
<link rel="profile" href="http://gmpg.org/xfn/11" />
<link rel="pingback" href="http://www.daveonfilm.com/xmlrpc.php" />
<title>Dave On Film: Smart Movie Reviews from Dave Taylor</title>
```

如果 `curl` 不可用，你可以通过 `lynx` 实现相同的结果，但如果你有两个工具，我们推荐使用 `curl`。这就是本章中我们将使用的工具。

**警告**

*本章中的网站抓取脚本的一个限制是，如果脚本依赖的某个网站在本书写作之后更改了其布局或 API，脚本可能会失效。但是，如果你能读取 HTML 或 JSON（即使你不完全理解它），你应该能够修复这些脚本。追踪其他网站的问题正是可扩展标记语言（XML）被创造出来的原因：它允许网站开发者将网页内容与其布局规则分开提供。*

### #53 通过 FTP 下载文件

互联网的原始杀手级应用之一是文件传输，而最简单的解决方案之一就是 FTP，即文件传输协议。从根本上说，所有的互联网交互都是基于文件传输的，无论是网页浏览器请求 HTML 文档及其附带的图像文件，聊天服务器来回传递讨论内容，还是电子邮件从地球的一端传递到另一端。

原始的 FTP 程序仍然存在，虽然它的界面简陋，但该程序功能强大、能力强，并且非常值得利用。现在有许多更新的 FTP 程序，特别是 FileZilla (*[`filezilla-project.org/`](http://filezilla-project.org/)*）和 NcFTP (*[`www.ncftp.org/`](http://www.ncftp.org/)*），以及很多你可以为 FTP 添加的漂亮图形界面，以使其更加用户友好。然而，借助一些 shell 脚本包装器，FTP 在上传和下载文件方面仍然表现得相当不错。

例如，FTP 的典型用例是从互联网上下载文件，这一点我们将在清单 7-1 中的脚本中实现。文件通常位于匿名 FTP 服务器上，且其 URL 类似于*ftp://<someserver>/<path>/<filename>*。

#### *代码*

```
   #!/bin/bash

   # ftpget--Given an ftp-style URL, unwraps it and tries to obtain the
   #   file using anonymous ftp

   anonpass="$LOGNAME@$(hostname)"

   if [ $# -ne 1 ] ; then
     echo "Usage: $0 ftp://..." >&2
     exit 1
   fi

   # Typical URL: ftp://ftp.ncftp.com/unixstuff/q2getty.tar.gz

   if [ "$(echo $1 | cut -c1-6)" != "ftp://" ] ; then
     echo "$0: Malformed url. I need it to start with ftp://" >&2
     exit 1
   fi

   server="$(echo $1 | cut -d/ -f3)"
   filename="$(echo $1 | cut -d/ -f4-)"
   basefile="$(basename $filename)"

   echo ${0}: Downloading $basefile from server $server

➊ ftp -np << EOF
   open $server
   user ftp $anonpass
   get "$filename" "$basefile"
   quit
   EOF

   if [ $? -eq 0 ] ; then
     ls -l $basefile
   fi

   exit 0
```

*清单 7-1：* `*ftpget*` *脚本*

#### *工作原理*

这个脚本的核心是从➊开始输入到 FTP 程序的一系列命令。这展示了批处理文件的本质：一系列指令被传递给一个独立的程序，让接收程序（在这个例子中是 FTP）认为这些指令是用户输入的。在这里，我们指定了要打开的服务器连接，指定了匿名用户（FTP）以及脚本配置中指定的默认密码（通常是你的电子邮件地址），然后从 FTP 站点获取指定的文件并退出传输。

#### *运行脚本*

这个脚本使用起来非常简单：只需要完全指定一个 FTP URL，它就会将文件下载到当前工作目录，正如清单 7-2 中所详细描述的那样。

#### *结果*

```
$ ftpget ftp://ftp.ncftp.com/unixstuff/q2getty.tar.gz
ftpget: Downloading q2getty.tar.gz from server ftp.ncftp.com
-rw-r--r--  1 taylor  staff  4817 Aug 14  1998 q2getty.tar.gz
```

*清单 7-2：运行* `*ftpget*` *脚本*

一些版本的 FTP 比其他版本更冗长，由于客户端和服务器协议有时会稍微不匹配，这些冗长版本的 FTP 可能会输出看起来很可怕的错误信息，如`Unimplemented command`。你可以安全地忽略这些错误。例如，清单 7-3 展示了相同的脚本在 OS X 上的运行结果。

```
$ ftpget ftp://ftp.ncftp.com/ncftp/ncftp-3.1.5-src.tar.bz2
../Scripts.new/053-ftpget.sh: Downloading q2getty.tar.gz from server ftp.
ncftp.com
Connected to ncftp.com.
220 ncftpd.com NcFTPd Server (licensed copy) ready.
331 Guest login ok, send your complete e-mail address as password.
230-You are user #2 of 16 simultaneous users allowed.
230-
230 Logged in anonymously.
Remote system type is UNIX.
Using binary mode to transfer files.
local: q2getty.tar.gz remote: unixstuff/q2getty.tar.gz
227 Entering Passive Mode (209,197,102,38,194,11)
150 Data connection accepted from 97.124.161.251:57849; transfer starting for
q2getty.tar.gz (4817 bytes).
100% |*******************************************************|  4817
67.41 KiB/s    00:00 ETA
226 Transfer completed.
4817 bytes received in 00:00 (63.28 KiB/s)
221 Goodbye.
-rw-r--r--  1 taylor  staff  4817 Aug 14  1998 q2getty.tar.gz
```

*清单 7-3：在 OS X 上运行* `*ftpget*` *脚本*

如果你的 FTP 过于冗长，并且你使用的是 OS X 系统，可以通过在脚本中为 FTP 调用添加`-V`标志来将其静音（也就是说，使用 FTP `-nV`，而不是 FTP `-n`）。

#### *修改脚本*

如果下载的文件具有某些文件扩展名，这个脚本可以扩展为自动解压下载的文件（参见脚本 #33，以及第 109 页中的示例，了解如何执行此操作）。许多压缩文件，如*.tar.gz*和*.tar.bz2*，默认可以使用系统的`tar`命令进行解压。

你还可以调整这个脚本，使其成为一个简单的工具，用于*上传*指定的文件到 FTP 服务器。如果服务器支持匿名连接（虽然如今很少有服务器支持，因为有脚本小子和其他不法分子，另当别论），你只需要在命令行或脚本中指定目标目录，并将主脚本中的`get`命令改为`put`，如下所示：

```
ftp -np << EOF
open $server
user ftp $anonpass
cd $destdir
put "$filename"
quit
EOF
```

要处理密码保护的账户，你可以通过在`read`语句之前关闭回显，然后在完成后再打开回显，来让脚本提示输入密码：

```
/bin/echo -n "Password for ${user}: "
stty -echo
read password
stty echo
echo ""
```

然而，更智能的密码提示方式是直接让 FTP 程序自己处理。这将在我们的脚本中实现，因为如果访问指定的 FTP 账户需要密码，FTP 程序会自动提示输入密码。

### #54 从网页提取 URL

`lynx` 的一种直接的 shell 脚本应用是提取指定网页上的 URL 列表，这在抓取互联网链接时非常有用。我们曾说过我们已经从 `lynx` 切换到 `curl` 用于本书的这一版本，但事实证明，对于这个脚本来说，`lynx` 使用起来要简单一百倍（参见 清单 7-4），因为 `lynx` 会自动解析 HTML，而 `curl` 需要你自己手动解析 HTML。

系统上没有 `lynx` 吗？如今大多数 Unix 系统都配有包管理器，如 Red Hat 上的 `yum`、Debian 上的 `apt` 以及 OS X 上的 `brew`（尽管 `brew` 默认没有安装），你可以使用它们来安装 `lynx`。如果你更喜欢自己编译 `lynx`，或者想下载预构建的二进制文件，可以从 *[`lynx.browser.org/`](http://lynx.browser.org/)* 下载。

#### *代码*

```
   #!/bin/bash

   # getlinks--Given a URL, returns all of its relative and absolute links.
   #   Has three options: -d to generate the primary domains of every link,
   #   -i to list just those links that are internal to the site (that is,
   #   other pages on the same site), and -x to produce external links only
   #   (the opposite of -i).

   if [ $# -eq 0 ] ; then
     echo "Usage: $0 [-d|-i|-x] url" >&2
     echo "-d=domains only, -i=internal refs only, -x=external only" >&2
     exit 1
   fi

   if [ $# -gt 1 ] ; then
     case "$1" in
➊     -d) lastcmd="cut -d/ -f3|sort|uniq"
           shift
           ;;
       -r) basedomain="http://$(echo $2 | cut -d/ -f3)/"
➋         lastcmd="grep \"^$basedomain\"|sed \"s|$basedomain||g\"|sort|uniq"
           shift
           ;;
       -a) basedomain="http://$(echo $2 | cut -d/ -f3)/"
➌         lastcmd="grep -v \"^$basedomain\"|sort|uniq"
           shift
           ;;
        *) echo "$0: unknown option specified: $1" >&2
           exit 1
     esac
   else
➍   lastcmd="sort|uniq"
   fi

   lynx -dump "$1"|\
➎   sed -n '/^References$/,$p'|\
     grep -E '[[:digit:]]+\.'|\
     awk '{print $2}'|\
     cut -d\? -f1|\
➏   eval $lastcmd

   exit 0
```

*清单 7-4：`*getlinks*` 脚本*

#### *它是如何工作的*

在显示页面时，`lynx` 会将页面的文本按其最佳方式格式化，接着显示该页面上找到的所有超文本引用或链接的列表。这个脚本通过使用 `sed` 命令提取网页文本中 `"References"` 字符串后的所有内容 ➎，然后根据用户指定的标志处理链接列表。

这个脚本展示的一个有趣技巧是如何通过设置变量 `lastcmd`（➊, ➋, ➌, ➍）来根据用户指定的标志筛选提取的链接列表。一旦设置了 `lastcmd`，就使用非常方便的 `eval` 命令 ➏ 强制 shell 将该变量的内容当作命令执行，而不是作为变量。

#### *运行脚本*

默认情况下，脚本会输出指定网页上找到的所有链接列表，而不仅仅是以 `http:` 开头的链接。不过，有三个可选的命令行标志可以指定以更改结果：`-d` 只输出所有匹配 URL 的域名，`-r` 输出仅包含 *相对* 引用的列表（即那些与当前页面位于同一服务器上的引用），`-a` 输出仅包含 *绝对* 引用的列表（即指向不同服务器的 URL）。

#### *结果*

一个简单的请求是列出指定网站主页上的所有链接，正如 清单 7-5 所示。

```
$ getlinks http://www.daveonfilm.com/ | head -10
http://instagram.com/d1taylor
http://pinterest.com/d1taylor/
http://plus.google.com/110193533410016731852
https://plus.google.com/u/0/110193533410016731852
https://twitter.com/DaveTaylor
http://www.amazon.com/Doctor-Who-Shada-Adventures-Douglas/
http://www.daveonfilm.com/
http://www.daveonfilm.com/about-me/
http://www.daveonfilm.com/author/d1taylor/
http://www.daveonfilm.com/category/film-movie-reviews/
```

*清单 7-5：运行 `*getlinks*` 脚本*

另一个可能的请求是列出特定网站上所有引用的域名。这次，让我们先使用标准的 Unix 工具 `wc` 来检查找到的链接总数：

```
$ getlinks http://www.amazon.com/ | wc -l
219
```

亚马逊首页上有 219 个链接。很令人印象深刻！这代表了多少个不同的域名呢？让我们使用`-d`标志生成一个列表：

```
$ getlinks -d http://www.amazon.com/ | head -10
amazonlocal.com
aws.amazon.com
fresh.amazon.com
kdp.amazon.com
services.amazon.com
www.6pm.com
www.abebooks.com
www.acx.com
www.afterschool.com
www.alexa.com
```

亚马逊通常不指向外部站点，但确实有一些合作伙伴链接会出现在主页上。当然，其他网站则不同。

如果我们将亚马逊页面上的链接分为相对链接和绝对链接，会怎么样？

```
$ getlinks -a http://www.amazon.com/ | wc -l
51
$ getlinks -r http://www.amazon.com/ | wc -l
222
```

正如你所预期的那样，亚马逊站点内部指向自己站点的相对链接比指向其他网站的绝对链接多四倍，这样做是为了让顾客始终停留在自己的网站上！

#### *破解脚本*

你可以看到`getlinks`作为站点分析工具是多么有用。为了增强脚本的功能，请关注：脚本 #69 在第 217 页很好地补充了这个脚本，使我们能够快速检查站点上的所有超文本引用是否有效。

### #55 获取 GitHub 用户信息

GitHub 已经成为开源行业和全球开放协作的巨大推动力。许多系统管理员和开发者访问 GitHub 来下载源代码或报告开源项目中的问题。由于 GitHub 本质上是一个面向开发者的社交平台，快速了解用户的基本信息非常有用。列表 7-6 中的脚本打印了关于某个 GitHub 用户的一些信息，并很好地介绍了功能强大的 GitHub API。

#### *代码*

```
   #!/bin/bash
   # githubuser--Given a GitHub username, pulls information about the user

   if [ $# -ne 1 ]; then
     echo "Usage: $0 <username>"
     exit 1
   fi

   # The -s silences curl's normally verbose output.
➊ curl -s "https://api.github.com/users/$1" | \
           awk -F'"' '
               /\"name\":/ {
                 print $4" is the name of the GitHub user."
               }
               /\"followers\":/{
                 split($3, a, " ")
                 sub(/,/, "", a[2])
                 print "They have "a[2]" followers."
               }
                 /\"following\":/{
                 split($3, a, " ")
                 sub(/,/, "", a[2])
                 print "They are following "a[2]" other users."
               }
               /\"created_at\":/{
                 print "Their account was created on "$4"."
               }
               '
   exit 0
```

*列表 7-6：* `*githubuser*` *脚本*

#### *它是如何工作的*

诚然，这几乎更像是一个`awk`脚本而不是 bash 脚本，但有时候你确实需要`awk`提供的额外功能来进行解析（GitHub API 返回的是 JSON 格式）。我们使用`curl`向 GitHub 请求用户➊信息，该信息作为脚本的参数，并将 JSON 数据传递给`awk`。在`awk`中，我们指定双引号字符作为字段分隔符，这样会使得解析 JSON 变得更简单。然后，我们使用几个正则表达式匹配 JSON 数据，并以用户友好的方式打印结果。

#### *运行脚本*

该脚本接受一个参数：在 GitHub 上查找的用户。如果提供的用户名不存在，则不会打印任何内容。

#### *结果*

当传入有效的用户名时，脚本应打印出一个用户友好的 GitHub 用户摘要，如列表 7-7 所示。

```
$ githubuser brandonprry
Brandon Perry is the name of the GitHub user.
They have 67 followers.
They are following 0 other users.
Their account was created on 2010-11-16T02:06:41Z.
```

*列表 7-7：运行* `*githubuser*` *脚本*

#### *破解脚本*

由于可以从 GitHub API 获取大量信息，这个脚本有很大的潜力。在这个脚本中，我们只打印了从 JSON 返回的四个值。基于 API 提供的信息为给定用户生成一份“简历”，就像许多网络服务所提供的那样，正是其中的一种可能性。

### #56 邮政编码查询

为了演示一种不同的网页抓取技术，这次我们使用`curl`，创建一个简单的邮政编码查询工具。给清单 7-8 中的脚本一个邮政编码，它会报告该邮政编码对应的城市和州。非常简单。

你最初的想法可能是使用美国邮政局的官方网站，但我们将使用另一个网站，*[`city-data.com/`](http://city-data.com/)*，它将每个邮政编码配置为一个独立的网页，因此信息提取起来更为简单。

#### *代码*

```
#!/bin/bash

# zipcode--Given a ZIP code, identifies the city and state. Use city-data.com,
#   which has every ZIP code configured as its own web page.

baseURL="http://www.city-data.com/zips"

/bin/echo -n "ZIP code $1 is in "

curl -s -dump "$baseURL/$1.html" | \
  grep -i '<title>' | \
  cut -d\( -f2 | cut -d\) -f1

exit 0
```

*清单 7-8：* `*zipcode*` *脚本*

#### *工作原理*

在*[`city-data.com/`](http://city-data.com/)*上，邮政编码信息页面的 URL 结构是一致的，邮政编码本身作为 URL 的最后一部分。

```
http://www.city-data.com/zips/80304.html
```

这种一致性使得为给定的邮政编码即时创建适当的 URL 变得非常容易。结果页面的标题中包含城市名称，并方便地用括号标明，格式如下。

```
<title>80304 Zip Code (Boulder, Colorado) Profile - homes, apartments,
schools, population, income, averages, housing, demographics, location,
statistics, residents and real estate info</title>
```

很长，但相当容易操作！

#### *运行脚本*

调用脚本的标准方法是在命令行中指定所需的邮政编码。如果它有效，将显示城市和州，如清单 7-9 所示。

#### *结果*

```
$ zipcode 10010
ZIP code 10010 is in New York, New York
$ zipcode 30001
ZIP code 30001 is in <title>Page not found – City-Data.com</title>
$ zipcode 50111
ZIP code 50111 is in Grimes, Iowa
```

*清单 7-9：运行* `*zipcode*` *脚本*

因为 30001 不是一个有效的邮政编码，脚本会生成一个`Page not found`错误。这有点草率，我们可以做得更好。

#### *破解脚本*

对这个脚本最明显的修改是，在遇到错误时做些处理，而不仅仅是输出那个丑陋的`<title>Page not found – City-Data.com</title>`序列。更有用的做法是添加一个`-a`标志，告诉脚本显示更多关于指定区域的信息，因为*[`city-data.com/`](http://city-data.com/)*提供了除城市名称外的很多信息——包括土地面积、人口统计以及房价。

### #57 区号查询

脚本 #56 中的邮政编码查询的变体是区号查询。这实际上非常简单，因为有一些非常易于解析的网页显示区号。位于*[`www.bennetyee.org/ucsd-pages/area.html`](http://www.bennetyee.org/ucsd-pages/area.html)*的页面尤其容易解析，不仅因为它是表格形式，还因为作者已经用 HTML 属性标识了元素。例如，定义区号 207 的那一行如下：

```
<tr><td align=center><a name="207">207</a></td><td align=center>ME</td><td
align=center>-5</td><td>   Maine</td></tr>
```

我们将使用这个网站查找在清单 7-10 中的脚本中的区号。

#### *代码*

```
#!/bin/bash

# areacode--Given a three-digit US telephone area code, identifies the city
#   and state using the simple tabular data at Bennet Yee's website.

source="http://www.bennetyee.org/ucsd-pages/area.html"

if [ -z "$1" ] ; then
  echo "usage: areacode <three-digit US telephone area code>"
  exit 1
fi

# wc -c returns characters + end of line char, so 3 digits = 4 chars
if [ "$(echo $1 | wc -c)" -ne 4 ] ; then
  echo "areacode: wrong length: only works with three-digit US area codes"
  exit 1
fi

# Are they all digits?
if [ ! -z "$(echo $1 | sed 's/[[:digit:]]//g')" ] ; then
  echo "areacode: not-digits: area codes can only be made up of digits"
  exit 1
fi

# Now, finally, let's look up the area code...

result="$(➊curl -s -dump $source | grep "name=\"$1" | \
  sed 's/<[^>]*>//g;s/^ //g' | \
  cut -f2- -d\ | cut -f1 -d\( )"

echo "Area code $1 =$result"

exit 0
```

*清单 7-10：* `*areacode*` *脚本*

#### *工作原理*

这个 Shell 脚本中的代码主要是输入验证，确保用户提供的数据是一个有效的区号。脚本的核心是一个`curl`调用 ➊，其输出通过管道传递给`sed`进行清理，然后使用`cut`裁剪成我们希望显示给用户的内容。

#### *运行脚本*

这个脚本接受一个参数，即要查询信息的区号。清单 7-11 展示了该脚本的使用示例。

#### *结果*

```
$ areacode 817
Area code 817 =  N Cent. Texas: Fort Worth area
$ areacode 512
Area code 512 =  S Texas: Austin
$ areacode 903
Area code 903 =  NE Texas: Tyler
```

*清单 7-11：测试* `*areacode*` *脚本*

#### *破解脚本*

一个简单的破解方法是反转搜索，提供州和城市名称，脚本则会打印给定城市的所有区号。

### #58 跟踪天气

长时间待在办公室或服务器房间里，面对终端工作，有时会让你渴望到外面走走，尤其是当天气特别好时。Weather Underground（* [`www.wunderground.com/`](http://www.wunderground.com/) *）是一个很棒的网站，实际上它为开发者提供了免费的 API，只要你注册一个 API 密钥。通过这个 API 密钥，我们可以编写一个快速的 Shell 脚本（如清单 7-12 所示），来告诉我们外面的天气有多好（或多差）。然后我们可以决定是否真的应该去散个步。

#### *代码*

```
   #!/bin/bash
   # weather--Uses the Wunderground API to get the weather for a given ZIP code

   if [ $# -ne 1 ]; then
     echo "Usage: $0 <zipcode>"
     exit 1
   fi

   apikey="b03fdsaf3b2e7cd23"   # Not a real API key--you need your own.

➊ weather=`curl -s \
       "https://api.wunderground.com/api/$apikey/conditions/q/$1.xml"`
➋ state=`xmllint --xpath \
       //response/current_observation/display_location/full/text\(\) \
       <(echo $weather)`
   zip=`xmllint --xpath \
       //response/current_observation/display_location/zip/text\(\) \
       <(echo $weather)`
   current=`xmllint --xpath \
       //response/current_observation/temp_f/text\(\) \
       <(echo $weather)`
   condition=`xmllint --xpath \
       //response/current_observation/weather/text\(\) \
       <(echo $weather)`

   echo $state" ("$zip") : Current temp "$current"F and "$condition" outside."

   exit 0
```

*清单 7-12：* `*weather*` *脚本*

#### *它是如何工作的*

在这个脚本中，我们使用`curl`调用 Wunderground API，并将 HTTP 响应数据保存在`weather`变量中➊。然后我们使用`xmllint`（可以通过你喜欢的包管理器，如`apt`、`yum`或`brew`轻松安装）工具对返回的数据执行 XPath 查询➋。我们还在调用`xmllint`时使用了一个有趣的 bash 语法，命令后面带有`<(echo $weather)`。这种语法将内部命令的输出传递给命令作为文件描述符，这样程序就认为它正在读取一个真实的文件。在从返回的 XML 中收集到所有相关信息后，我们会打印一条友好的消息，显示天气的基本统计信息。

#### *运行脚本*

当你调用脚本时，只需指定所需的邮政编码，如清单 7-13 所示。非常简单！

#### *结果*

```
$ weather 78727
Austin, TX (78727) : Current temp 59.0F and Clear outside.
$ weather 80304
Boulder, CO (80304) : Current temp 59.2F and Clear outside.
$ weather 10010
New York, NY (10010) : Current temp 68.7F and Clear outside.
```

*清单 7-13：测试* `*weather*` *脚本*

#### *破解脚本*

我们有个小秘密。这个脚本实际上可以接受的不仅仅是邮政编码。你还可以在 Wunderground API 中指定区域，比如`CA/San_Francisco`（尝试作为天气脚本的参数！）。然而，这种格式并不是非常用户友好：它要求用下划线代替空格，并且中间有一个斜杠。如果能够添加一个功能，允许用户输入州的缩写和城市名，并在没有传入参数时将空格替换为下划线，那会是一个有用的改进。和往常一样，这个脚本还可以加上更多的错误检查代码。如果你输入了一个四位数的邮政编码会发生什么？或者一个未分配的邮政编码呢？

### #59 从 IMDb 挖掘电影信息

清单 7-14 中的脚本演示了通过`lynx`访问互联网的一种更复杂方式，通过搜索互联网电影数据库（* [`www.imdb.com/`](http://www.imdb.com/) *）查找与指定模式匹配的电影。IMDb 为每部电影、电视系列以及甚至每一集电视剧分配了唯一的数字代码；如果用户指定了该代码，脚本将返回电影的简介。否则，它会根据标题或部分标题返回匹配的电影列表。

脚本根据查询类型（数字 ID 或文件标题）访问不同的 URL，并缓存结果，以便它可以多次浏览页面，提取不同的信息。而且它使用了很多——*很多*——`sed`和`grep`的调用，正如你将看到的那样。

#### *代码*

```
   #!/bin/bash
   # moviedata--Given a movie or TV title, returns a list of matches. If the user
   #   specifies an IMDb numeric index number, however, returns the synopsis of
   #   the film instead. Uses the Internet Movie Database.

   titleurl="http://www.imdb.com/title/tt"
   imdburl="http://www.imdb.com/find?s=tt&exact=true&ref_=fn_tt_ex&q="
   tempout="/tmp/moviedata.$$"

➊ summarize_film()
   {
     # Produce an attractive synopsis of the film.

     grep "<title>" $tempout | sed 's/<[^>]*>//g;s/(more)//'

     grep --color=never -A2 '<h5>Plot:' $tempout | tail -1 | \
       cut -d\< -f1 | fmt | sed 's/^/ /'

     exit 0
   }

   trap "rm -f $tempout" 0 1 15

   if [ $# -eq 0 ] ; then
     echo "Usage: $0 {movie title | movie ID}" >&2
     exit 1
   fi

   #########
   # Checks whether we're asking for a title by IMDb title number

   nodigits="$(echo $1 | sed 's/[[:digit:]]*//g')"

   if [ $# -eq 1 -a -z "$nodigits" ] ; then
     lynx -source "$titleurl$1/combined" > $tempout
     summarize_film
     exit 0
   fi

   ##########
   # It's not an IMDb title number, so let's go with the search...

   fixedname="$(echo $@ | tr ' ' '+')"       # for the URL

   url="$imdburl$fixedname"

➋ lynx -source $imdburl$fixedname > $tempout

   # No results?

➌ fail="$(grep --color=never '<h1 class="findHeader">No ' $tempout)"

   # If there's more than one matching title...

   if [ ! -z "$fail" ] ; then
     echo "Failed: no results found for $1"
     exit 1
   elif [ ! -z "$(grep '<h1 class="findHeader">Displaying' $tempout)" ] ; then
     grep --color=never '/title/tt' $tempout | \
     sed 's/</\
   </g' | \
     grep -vE '(.png|.jpg|>[ ]*$)' | \
     grep -A 1 "a href=" | \
     grep -v '^--$' | \
     sed 's/<a href="\/title\/tt//g;s/<\/a> //' | \
➍   awk '(NR % 2 == 1) { title=$0 } (NR % 2 == 0) { print title " " $0 }' | \
     sed 's/\/.*>/: /' | \
     sort
   fi

   exit 0
```

*清单 7-14：* `*moviedata*` *脚本*

#### *它是如何工作的*

此脚本根据命令参数指定的是电影标题还是 IMDb ID 号码来构建不同的 URL。如果用户通过 ID 号码指定标题，脚本会构建适当的 URL，下载它，将`lynx`输出保存到`$tempout`文件 ➋ 中，并最终调用`summarize_film()` ➊。并不难。

但如果用户指定了标题，则脚本将为 IMDb 上的搜索查询构建一个 URL，并将结果页面保存到临时文件中。如果 IMDb 找不到匹配项，则返回的 HTML 中`<h1>`标签的`class="findHeader"`值将显示`没有结果`。这就是在 ➌ 中检查的内容。然后，测试很简单：如果`$fail`的长度不为零，脚本可以报告未找到任何结果。

然而，如果结果*是*零长度，这意味着`$tempfile`现在包含一个或多个成功的搜索结果，这些结果符合用户的模式。可以通过在源代码中搜索`/title/tt`作为模式来提取这些结果，但有个警告：IMDb 并没有使解析结果变得容易，因为任何给定的标题链接都有多个匹配项。其余的`sed|grep|sed`序列试图识别并移除重复的匹配项，同时保留那些重要的匹配项。

此外，当 IMDb 匹配到类似`"阿拉伯的劳伦斯 (1962)"`的条目时，标题和年份实际上是两个不同的 HTML 元素，分别位于结果的两行中。呃。我们需要年份来区分同一标题但在不同年份上映的电影。这就是 ➍ 中的`awk`语句所做的事情，以一种巧妙的方式。

如果你不熟悉`awk`，`awk`脚本的一般格式是`(*condition*) { *action* }`。这行代码将奇数行的数据保存在`$title`中，然后在偶数行（年份和匹配类型数据）中，它将前一行和当前行的数据作为一行输出。

#### *运行脚本*

尽管该脚本较短，但它在输入格式方面相当灵活，正如在列表 7-15 中所示。你可以用引号指定电影标题或作为单独的单词输入，然后你可以指定八位数的 IMDb ID 值来选择特定的匹配项。

#### *结果*

```
$ moviedata lawrence of arabia
0056172: Lawrence of Arabia (1962)
0245226: Lawrence of Arabia (1935)
0390742: Mighty Moments from World History (1985) (TV Series)
1471868: Mystery Files (2010) (TV Series)
1471868: Mystery Files (2010) (TV Series)
1478071: Lawrence of Arabia (1985) (TV Episode)
1942509: Lawrence of Arabia (TV Episode)
1952822: Lawrence of Arabia (2011) (TV Episode)
$ moviedata 0056172
Lawrence of Arabia (1962)
    A flamboyant and controversial British military figure and his
    conflicted loyalties during his World War I service in the Middle East.
```

*列表 7-15：运行* `*moviedata*` *脚本*

#### *破解脚本*

对这个脚本最明显的修改是去掉输出中难看的 IMDb 电影 ID 编号。隐藏电影 ID（因为显示的 ID 相当不友好且容易出错）并让 shell 脚本输出一个简单的菜单，其中包含唯一的索引值，然后可以输入这些值来选择特定的电影，应该是简单的。

在确切匹配到一部电影的情况下（试试 `moviedata monsoon wedding`），如果脚本能识别出这是唯一的匹配项，抓取电影编号并重新调用自己获取数据，那就太好了。试试看！

这个脚本的问题，与大多数从第三方网站抓取数据的脚本一样，如果 IMDb 更改了页面布局，脚本就会失效，你需要重新构建脚本顺序。这是一个潜在的错误，但对于像 IMDb 这样多年来没有变化的网站来说，可能并不是一个危险问题。

### #60 计算货币值

在本书的第一版中，货币转换是一个相当困难的任务，需要两个脚本：一个从金融网站获取转换汇率并以特定格式保存，另一个使用这些数据实际进行转换——比如将美元转换为欧元。然而，在这几年里，互联网变得更加复杂，我们无需再进行大量的工作，因为像 Google 这样的站点提供了简单、适合脚本使用的计算器。

对于这个版本的货币转换脚本，如列表 7-16 所示，我们将直接使用 *[`www.google.com/finance/converter`](http://www.google.com/finance/converter)* 中的货币计算器。

#### *代码*

```
#!/bin/bash

# convertcurrency--Given an amount and base currency, converts it
#   to the specified target currency using ISO currency identifiers.
#   Uses Google's currency converter for the heavy lifting:
#   http://www.google.com/finance/converter

if [ $# -eq 0 ]; then
  echo "Usage: $(basename $0) amount currency to currency"
  echo "Most common currencies are CAD, CNY, EUR, USD, INR, JPY, and MXN"
  echo "Use \"$(basename $0) list\" for a list of supported currencies."
fi

if [ $(uname) = "Darwin" ]; then
  LANG=C   # For an issue on OS X with invalid byte sequences and lynx
fi

     url="https://www.google.com/finance/converter"
tempfile="/tmp/converter.$$"
    lynx=$(which lynx)

# Since this has multiple uses, let's grab this data before anything else.

currencies=$($lynx -source "$url" | grep "option value=" | \
  cut -d\" -f2- | sed 's/">/ /' | cut -d\( -f1 | sort | uniq)

########### Deal with all non-conversion requests.

if [ $# -ne 4 ] ; then
  if [ "$1" = "list" ] ; then
    # Produce a listing of all currency symbols known by the converter.
    echo "List of supported currencies:"
    echo "$currencies"
  fi
  exit 0
fi

########### Now let's do a conversion.

if [ $3 != "to" ] ; then
  echo "Usage: $(basename $0) value currency TO currency"
  echo "(use \"$(basename $0) list\" to get a list of all currency values)"
  exit 0
fi

amount=$1
basecurrency="$(echo $2 | tr '[:lower:]' '[:upper:]')"
targetcurrency="$(echo $4 | tr '[:lower:]' '[:upper:]')"

# And let's do it--finally!

$lynx -source "$url?a=$amount&from=$basecurrency&to=$targetcurrency" | \
  grep 'id=currency_converter_result' | sed 's/<[^>]*>//g'

exit 0
```

*列表 7-16：* `*convertcurrency*` *脚本*

#### *工作原理*

Google 货币转换器有三个通过 URL 传递的参数：金额、原始货币和你想转换成的货币。你可以在以下请求中看到它是如何工作的，将 100 美元转换为墨西哥比索。

```
https://www.google.com/finance/converter?a=100&from=USD&to=MXN
```

在最基本的使用案例中，脚本期望用户指定这三个字段作为参数，然后将它们通过 URL 传递给 Google。

脚本还提供了一些使用消息，便于使用。为了看到这些信息，我们不妨直接跳到演示部分，怎么样？

#### *运行脚本*

这个脚本的设计目标是易于使用，正如列表 7-17 中详细描述的那样，尽管至少对一些国家的货币有基本了解会更有帮助。

#### *结果*

```
$ convertcurrency
Usage: convert amount currency to currency
Most common currencies are CAD, CNY, EUR, USD, INR, JPY, and MXN
Use "convertcurrency list" for a list of supported currencies.
$ convertcurrency list | head -10
List of supported currencies:

AED United Arab Emirates Dirham
AFN Afghan Afghani
ALL Albanian Lek
AMD Armenian Dram
ANG Netherlands Antillean Guilder
AOA Angolan Kwanza
ARS Argentine Peso
AUD Australian Dollar
AWG Aruban Florin
$ convertcurrency 75 eur to usd
75 EUR = 84.5132 USD
```

*列表 7-17：运行* `*convertcurrency*` *脚本*

#### *破解脚本*

虽然这个基于网页的计算器简单易用，但输出结果可以进行一些整理。例如，示例 7-17 中的输出并不完全合理，因为它用四位小数表示美元，而实际上美分只有两位小数。正确的输出应该是 84.51，或者四舍五入后为 84.52。这是脚本中可以修正的部分。

在此基础上，验证货币缩写会很有帮助。同样，将这些货币代码转化为完整的货币名称也是一个不错的功能，这样你就能知道 AWG 是阿鲁巴弗罗林，BTC 是比特币。

### #61 获取比特币地址信息

比特币已经席卷全球，围绕着*区块链*技术（比特币工作的核心）建立了许多企业。对于任何使用比特币的人来说，获取特定比特币地址的有用信息可能是一个麻烦。但是，我们可以通过快速的 shell 脚本轻松自动化数据收集，像示例 7-18 中所展示的那样。

#### *代码*

```
#!/bin/bash
# getbtcaddr--Given a Bitcoin address, reports useful information

if [ $# -ne 1 ]; then
  echo "Usage: $0 <address>"
  exit 1
fi

base_url="https://blockchain.info/q/"

balance=$(curl -s $base_url"addressbalance/"$1)
recv=$(curl -s $base_url"getreceivedbyaddress/"$1)
sent=$(curl -s $base_url"getsentbyaddress/"$1)
first_made=$(curl -s $base_url"addressfirstseen/"$1)

echo "Details for address $1"
echo -e "\tFirst seen: "$(date -d @$first_made)
echo -e "\tCurrent balance: "$balance
echo -e "\tSatoshis sent: "$sent
echo -e "\tSatoshis recv: "$recv
```

*示例 7-18：* `*getbtcaddr*` *脚本*

#### *工作原理*

该脚本自动化了一些`curl`调用，以检索给定比特币地址的几个关键信息。* [`blockchain.info/`](http://blockchain.info/)*提供的 API 让我们非常方便地访问各种比特币和区块链信息。实际上，我们甚至不需要解析从 API 返回的响应，因为它仅返回单一的、简单的值。在获取给定地址的余额、已发送和已接收的 BTC 数量以及创建时间后，脚本将信息打印到屏幕上供用户查看。

#### *运行脚本*

该脚本只接受一个参数，即我们想要获取信息的比特币地址。然而，我们需要提到，如果传入的字符串不是一个有效的比特币地址，脚本将仅打印所有余额、已发送和已接收的值为 0，并且创建日期会显示为 1969 年。任何非零值都以*satoshis*为单位，satoshi 是比特币的最小单位（类似于美分，但小数点后位数更多）。

#### *结果*

运行`getbtcaddr` shell 脚本非常简单，只需要一个参数——请求数据的比特币地址，正如示例 7-19 所示。

```
$ getbtcaddr 1A1zP1eP5QGefi2DMPTfTL5SLmv7DivfNa
Details for address 1A1zP1eP5QGefi2DMPTfTL5SLmv7DivfNa
    First seen: Sat Jan 3 12:15:05 CST 2009
    Current balance: 6554034549
    Satoshis sent: 0
    Satoshis recv: 6554034549

$ getbtcaddr 1EzwoHtiXB4iFwedPr49iywjZn2nnekhoj
Details for address 1EzwoHtiXB4iFwedPr49iywjZn2nnekhoj
    First seen: Sun Mar 11 11:11:41 CDT 2012
    Current balance: 2000000
    Satoshis sent: 716369585974
    Satoshis recv: 716371585974
```

*示例 7-19：运行* `*getbtcaddr*` *脚本*

#### *修改脚本*

默认情况下，屏幕上打印的数字相当大，对于大多数人来说有些难以理解。可以很容易地使用`scriptbc`脚本（脚本 #9，在第 34 页）以更合理的单位报告信息，比如整个比特币。为脚本添加一个比例参数将是让用户获得更易读的输出的一种简单方法。

### #62 跟踪网页变化

有时，伟大的灵感来自于看到一个现有的业务并对自己说：“这似乎并不难。”在网站上跟踪变化的任务，实际上是收集这种灵感材料的一种出奇简单的方法。清单 7-20 中的脚本`changetrack`自动化了这一任务。这个脚本有一个有趣的细节：当它检测到站点发生变化时，它会将新网页发送给用户，而不是仅仅在命令行上报告信息。

#### *代码*

```
   #!/bin/bash

   # changetrack--Tracks a given URL and, if it's changed since the last visit,
   #   emails the new page to the specified address

   sendmail=$(which sendmail)
   sitearchive="/tmp/changetrack"
   tmpchanges="$sitearchive/changes.$$"  # Temp file
   fromaddr="webscraper@intuitive.com"
   dirperm=755        # read+write+execute for dir owner
   fileperm=644       # read+write for owner, read only for others

   trap "$(which rm) -f $tmpchanges" 0 1 15  # Remove temp file on exit

   if [ $# -ne 2 ] ; then
     echo "Usage: $(basename $0) url email" >&2
     echo "  tip: to have changes displayed on screen, use email addr '-'" >&2
     exit 1
   fi

   if [ ! -d $sitearchive ] ; then
     if ! mkdir $sitearchive ; then
 echo "$(basename $0) failed: couldn't create $sitearchive." >&2
       exit 1
     fi
     chmod $dirperm $sitearchive
   fi

   if [ "$(echo $1 | cut -c1-5)" != "http:" ] ; then
     echo "Please use fully qualified URLs (e.g. start with 'http://')" >&2
     exit 1
   fi

   fname="$(echo $1 | sed 's/http:\/\///g' | tr '/?&' '...')"
   baseurl="$(echo $1 | cut -d/ -f1-3)/"

   # Grab a copy of the web page and put it in an archive file. Note that we
   #   can track changes by looking just at the content (that is, -dump, not
   #   -source), so we can skip any HTML parsing....

   lynx -dump "$1" | uniq > $sitearchive/${fname}.new
   if [ -f "$sitearchive/$fname" ] ; then
     # We've seen this site before, so compare the two with diff.
     diff $sitearchive/$fname $sitearchive/${fname}.new > $tmpchanges
     if [ -s $tmpchanges ] ; then
       echo "Status: Site $1 has changed since our last check."
     else
       echo "Status: No changes for site $1 since last check."
       rm -f $sitearchive/${fname}.new     # Nothing new...
       exit 0                              # No change--we're outta here.
     fi
   else
     echo "Status: first visit to $1\. Copy archived for future analysis."
     mv $sitearchive/${fname}.new $sitearchive/$fname
     chmod $fileperm $sitearchive/$fname
     exit 0
   fi

   # If we're here, the site has changed, and we need to send the contents
   #   of the .new file to the user and replace the original with the .new
   #   for the next invocation of the script.

   if [ "$2" != "-" ] ; then

   ( echo "Content-type: text/html"
     echo "From: $fromaddr (Web Site Change Tracker)"
     echo "Subject: Web Site $1 Has Changed"
➊   echo "To: $2"
     echo ""

➋   lynx -s -dump $1 | \
➌   sed -e "s|src=\"|SRC=\"$baseurl|gi" \
➍       -e "s|href=\"|HREF=\"$baseurl|gi" \
➎       -e "s|$baseurl\/http:|http:|g"
   ) | $sendmail -t

   else
     # Just showing the differences on the screen is ugly. Solution?

     diff $sitearchive/$fname $sitearchive/${fname}.new
   fi

   # Update the saved snapshot of the website.

   mv $sitearchive/${fname}.new $sitearchive/$fname
   chmod 755 $sitearchive/$fname
   exit 0
```

*清单 7-20：`*changetrack*`*脚本*

#### *工作原理*

给定一个 URL 和目标电子邮件地址，脚本会获取网页内容并与上次检查时的网站内容进行比较。如果站点发生了变化，新网页会通过电子邮件发送给指定的接收人，并对图形和`href`标签进行一些简单的重写，尽力保持它们正常工作。这个从➋开始的 HTML 重写值得一看。

调用`lynx`命令获取指定网页的源代码➋，然后`sed`执行三种不同的翻译。首先，`SRC="`被重写为`SRC="baseurl/`➌，确保任何相对路径名（如`SRC="logo.gif"`）都会被重写为带有域名的完整路径名。如果站点的域名是*[`www.intuitive.com/`](http://www.intuitive.com/)*，重写后的 HTML 将是`SRC="http://www.intuitive.com/logo.gif"`。同样，`href`属性也会被重写➍。然后，为了确保没有破坏任何内容，第三次翻译会在误加了`baseurl`的 HTML 源中将其移除➎。例如，`HREF="http://www.intuitive.com/http://www.somewhereelse.com/link"`显然是错误的，必须修复才能使链接正常工作。

还需要注意的是，接收地址是在`echo`语句➊（`echo "To: $2"）`中指定的，而不是作为`sendmail`的参数。这是一个简单的安全技巧：通过将地址放在`sendmail`的输入流中（`sendmail`会根据`-t`标志知道解析收件人），就不用担心用户恶意篡改地址，例如`"joe;cat /etc/passwd|mail larry"`。在使用`sendmail`时，这是一个很好的安全实践。

#### *运行脚本*

这个脚本需要两个参数：被跟踪站点的 URL（为了正常工作，你需要使用以`http://`开头的完整 URL）以及应该接收更新网页的人员的电子邮件地址（或者以逗号分隔的多人邮件地址）。或者，如果你更喜欢，可以将电子邮件地址设为`-`（一个连字符），那么`diff`输出将显示在屏幕上。

#### *结果*

第一次运行脚本时，网页会自动通过电子邮件发送给指定的用户，如清单 7-21 所示。

```
$ changetrack http://www.intuitive.com/ taylor@intuitive.com
Status: first visit to http://www.intuitive.com/. Copy archived for future
analysis.
```

*清单 7-21：第一次运行*`*changetrack*`*脚本*

对* [`www.intuitive.com/`](http://www.intuitive.com/)*的所有后续检查，将只在页面自上次脚本调用以来发生变化时，才会发送该网站的电子邮件副本。这一变化可以是一个简单的拼写错误修复，也可以是一个完整的重新设计。虽然这个脚本可以用于跟踪任何网站，但不经常变化的网站可能效果最佳：如果该网站是 BBC 新闻主页，检查变化就浪费 CPU 周期，因为这个网站是*不断*更新的。

如果第二次调用脚本时，网站没有发生变化，脚本将没有输出，并且不会向指定的接收人发送电子邮件：

```
$ changetrack http://www.intuitive.com/ taylor@intuitive.com
$
```

#### *破解脚本*

当前脚本的一个明显缺陷是它硬编码为查找*http://*链接，这意味着它会拒绝任何通过 HTTPS 和 SSL 提供的 HTTP 网页。更新脚本以同时支持两者将需要一些更复杂的正则表达式，但完全是可能的！

另一个使脚本更有用的改动是添加一个粒度选项，允许用户指定如果只有一行发生变化，脚本不应该认为该网站已经更新。你可以通过将`diff`输出传递给`wc -l`来实现这一点，以统计输出发生变化的行数。（记住，`diff`通常会为每行变化输出*三*行内容。）

当这个脚本从`cron`任务中按日或每周调用时，它也更加有用。我们有类似的脚本每天晚上运行，向我们发送来自各种网站的更新网页，这些网站是我们喜欢跟踪的。

一个特别有趣的可能性是修改这个脚本，使其能够处理一个包含网址和电子邮件地址的数据文件，而不需要将这些作为输入参数。将修改后的脚本放入`cron`任务中，编写一个基于网页的前端工具（类似于第八章中的 Shell 脚本），你就复制了一个一些公司收费使用的功能。不是开玩笑。
