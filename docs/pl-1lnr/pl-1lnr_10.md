## 附录 C. Perl1Line.Txt

当我写这本书时，我将所有的单行代码汇编到一个名为*perl1line.txt*的文件中。本附录就是该文件。当你需要快速查找一个单行代码时，它非常方便。你只需在文本编辑器中打开*perl1line.txt*并搜索你想要执行的操作。此文件的最新版本可以在* [`www.catonmat.net/download/perl1line.txt`](http://www.catonmat.net/download/perl1line.txt) *找到。

## C.1 间距

#### 双倍行距一个文件

```
perl -pe '$\ = "\n"'
perl -pe 'BEGIN { $\ = "\n" }'
perl -pe '$_ .= "\n"'
perl -pe 's/$/\n/'
perl -nE 'say'
```

#### 双倍行距一个文件，排除空白行

```
perl -pe '$_ .= "\n" unless /^$/'
perl -pe '$_ .= "\n" if /\S/'
```

#### 三倍行距一个文件

```
perl -pe '$\ = "\n\n"'
perl -pe '$_ .= "\n\n"'
perl -pe 's/$/\n\n/'
```

#### N-space 一个文件

```
perl -pe '$_ .= "\n"x7'
```

#### 每行前添加一个空白行

```
perl -pe 's/^/\n/'
```

#### 删除所有空白行

```
perl -ne 'print unless /^$/'
perl -lne 'print if length'
perl -ne 'print if /\S/'
```

#### 删除所有连续的空白行，仅保留一行

```
perl -00 -pe ''
perl -00pe0
```

#### 压缩/展开所有空白行为 N 个连续的行

```
perl -00 -pe '$_ .= "\n"x2'
```

#### 在所有单词之间双倍行距

```
perl -pe 's/ /  /g'
```

#### 删除所有单词之间的间距

```
perl -pe 's/ +//g'
perl -pe 's/\s+//g'
```

#### 将所有单词之间的间距改为一个空格

```
perl -pe 's/ +/ /g'
```

#### 在所有字符之间插入一个空格

```
perl -lpe 's// /g'
```

## C.2 编号

#### 给文件中的所有行编号

```
perl -pe '$_ = "$. $_"'
perl -ne 'print "$. $_"'
```

#### 仅给文件中非空行编号

```
perl -pe '$_ = ++$x." $_" if /./'
perl -pe '$_ = ++$x." $_" if /\S/'
```

#### 给文件中的非空行编号并打印（删除空行）

```
perl -ne 'print ++$x." $_" if /./'
```

#### 给所有行编号，但仅对非空行打印行号

```
perl -pe '$_ = "$. $_" if /./'
```

#### 只给匹配模式的行编号；其他行保持不变

```
perl -pe '$_ = ++$x." $_" if /regex/'
```

#### 仅对匹配模式的行进行编号和打印

```
perl -ne 'print ++$x." $_" if /regex/'
```

#### 给所有行编号，但仅对匹配模式的行打印行号

```
perl -pe '$_ = "$. $_" if /regex/'
```

#### 使用自定义格式给文件中的所有行编号

```
perl -ne 'printf "%-5d %s", $., $_'
```

#### 打印文件中的总行数（模拟 wc -l）

```
perl -lne 'END { print $. }'
perl -le 'print $n = () = <>'
perl -le 'print $n = (() = <>)'
perl -le 'print scalar(() = <>)'
perl -le 'print scalar(@foo = <>)'
perl -ne '}{print $.'
```

#### 打印文件中非空行的数量

```
perl -le 'print scalar(grep { /./ } <>)'
perl -le 'print ~~grep{/./}<>'
perl -le 'print~~grep/./,<>'
perl -lE 'say~~grep/./,<>'
```

#### 打印文件中空白行的数量

```
perl -lne '$x++ if /^$/; END { print $x+0 }'
perl -lne '$x++ if /^$/; END { print int $x }'
perl -le 'print scalar(grep { /^$/ } <>)'
perl -le 'print ~~grep{ /^$/ } <>'
```

#### 打印匹配模式的行数（模拟 grep -c）

```
perl -lne '$x++ if /regex/; END { print $x+0 }'
```

#### 计算所有行的单词数

```
perl -pe 's/(\w+)/++$i.".$1"/ge'
```

#### 给每行的单词编号

```
perl -pe '$i=0; s/(\w+)/++$i.".$1"/ge'
```

#### 用单词的数字位置替换所有单词

```
perl -pe 's/(\w+)/++$i/ge'
```

## C.3 计算

#### 检查一个数字是否是质数

```
perl -lne '(1x$_) !~ /¹?$|^(11+?)\1+$/ && print "$_ is prime"'
```

#### 打印每行所有字段的总和

```
perl -MList::Util=sum -alne 'print sum @F'
perl -MList::Util=sum -F: -alne 'print sum @F'
```

#### 打印所有行中字段的总和

```
perl -MList::Util=sum -alne 'push @S,@F; END { print sum @S }'
perl -MList::Util=sum -alne '$s += sum @F; END { print $s }'
```

#### 打乱每行上的所有字段

```
perl -MList::Util=shuffle -alne 'print "@{[shuffle @F]}"'
perl -MList::Util=shuffle -alne 'print join " ", shuffle @F'
```

#### 查找每行中数值最小的元素（最小元素）

```
perl -MList::Util=min -alne 'print min @F'
```

#### 查找所有行中数值最小的元素（最小元素）

```
perl -MList::Util=min -alne '@M = (@M, @F); END { print min @M }'
perl -MList::Util=min -alne '
  $min = min @F;
  $rmin = $min unless defined $rmin && $min > $rmin;
  END { print $rmin }
'
perl -MList::Util=min -alne '$min = min($min // (), @F); END { print $min }'
```

#### 查找每行中数值最大的元素（最大元素）

```
perl -MList::Util=max -alne 'print max @F'
```

#### 查找所有行中数值最大的元素（最大元素）

```
perl -MList::Util=max -alne '@M = (@M, @F); END { print max @M }'
perl -MList::Util=max -alne '
  $max = max @F;
  $rmax = $max unless defined $rmax && $max < $rmax;
  END { print $rmax }
'
perl -MList::Util=max -alne '$max = max($max // (), @F); END { print $max }'
```

#### 将每个字段替换为其绝对值

```
perl -alne 'print "@{[map { abs } @F]}"'
```

#### 打印每行的字段总数

```
perl -alne 'print scalar @F'
```

#### 打印每行的字段总数，后跟该行

```
perl -alne 'print scalar @F, " $_"'
```

#### 打印所有行中字段的总数

```
perl -alne '$t += @F; END { print $t }'
```

#### 打印与模式匹配的字段的总数

```
perl -alne 'map { /regex/ && $t++ } @F; END { print $t || 0 }'
perl -alne '$t += /regex/ for @F; END { print $t }'
perl -alne '$t += grep /regex/, @F; END { print $t }'
```

#### 打印与模式匹配的行的总数

```
perl -lne '/regex/ && $t++; END { print $t || 0 }'
```

#### 打印数字π

```
perl -Mbignum=bpi -le 'print bpi(21)'
perl -Mbignum=PI -le 'print PI'
```

#### 打印数字 *e*

```
perl -Mbignum=bexp -le 'print bexp(1,21)'
perl -Mbignum=e -le 'print e'
```

#### 打印 UNIX 时间（自 1970 年 1 月 1 日 00:00:00 UTC 以来的秒数）

```
perl -le 'print time'
```

#### 打印格林威治标准时间和本地计算机时间

```
perl -le 'print scalar gmtime'
perl -le 'print scalar localtime'
```

#### 打印昨天的日期

```
perl -MPOSIX -le '
  @now = localtime;
  $now[3] -= 1;
  print scalar localtime mktime @now
'
```

#### 打印 14 个月 9 天 7 秒前的日期

```
perl -MPOSIX -le '
  @now = localtime;
  $now[0] -= 7;
  $now[3] -= 9;
  $now[4] -= 14;
  print scalar localtime mktime @now
'
```

#### 计算阶乘

```
perl -MMath::BigInt -le 'print Math::BigInt->new(5)->bfac()'
perl -le '$f = 1; $f *= $_ for 1..5; print $f'
```

#### 计算最大公约数

```
perl -MMath::BigInt=bgcd -le 'print bgcd(@list_of_numbers)'
perl -MMath::BigInt=bgcd -le 'print bgcd(20,60,30)'
perl -MMath::BigInt=bgcd -anle 'print bgcd(@F)'
perl -le '
  $n = 20; $m = 35;
  ($m,$n) = ($n,$m%$n) while $n;
  print $m
'
```

#### 计算最小公倍数

```
perl -MMath::BigInt=blcm -le 'print blcm(35,20,8)'
perl -MMath::BigInt=blcm -anle 'print blcm(@F)'
perl -le '
  $a = $n = 20;
  $b = $m = 35;
  ($m,$n) = ($n,$m%$n) while $n;
  print $a*$b/$m
'
```

#### 生成 10 个 5 到 15 之间的随机数（不包括 15）

```
perl -le 'print join ",", map { int(rand(15-5))+5 } 1..10'
perl -le '
  $n=10;
  $min=5;
  $max=15;
  $, = " ";
  print map { int(rand($max-$min))+$min } 1..$n;
'
```

#### 生成一个列表的所有排列

```
perl -MAlgorithm::Permute -le '
  $l = [1,2,3,4,5];
  $p = Algorithm::Permute->new($l);
  print "@r" while @r = $p->next
'
 perl -MAlgorithm::Permute -le '
  @l = (1,2,3,4,5);
  Algorithm::Permute::permute { print "@l" } @l
'
```

#### 生成幂集

```
perl -MList::PowerSet=powerset -le '
  @l = (1,2,3,4,5);
  print "@$_" for @{powerset(@l)}
'
```

#### 将 IP 地址转换为无符号整数

```
perl -le '
  $i=3;
  $u += ($_<<8*$i--) for "127.0.0.1" =~ /(\d+)/g;
  print $u
'
 perl -le '
  $ip="127.0.0.1";
  $ip =~ s/(\d+)\.?/sprintf("%02x", $1)/ge;
  print hex($ip)
'
 perl -le 'print unpack("N", 127.0.0.1)'
 perl -MSocket -le 'print unpack("N", inet_aton("127.0.0.1"))'
```

#### 将无符号整数转换为 IP 地址

```
perl -MSocket -le 'print inet_ntoa(pack("N", 2130706433))'
perl -le '
  $ip = 2130706433;
  print join ".", map { (($ip>>8*($_))&0xFF) } reverse 0..3
'
perl -le '
  $ip = 2130706433;
  $, = ".";
  print map { (($ip>>8*($_))&0xFF) } reverse 0..3
'
perl -le '
  $ip = 2130706433;
  $, = ".";
  print map { (($ip>>8*($_))&0xFF) } 3,2,1,0
'
```

## C.4 处理数组和字符串

#### 生成并打印字母表

```
perl -le 'print a..z'
perl -le 'print ("a".."z")'
perl -le '$, = ","; print ("a".."z")'
perl -le '$alphabet = join ",", ("a".."z"); print $alphabet'
```

#### 生成并打印从“a”到“zz”的所有字符串

```
perl -le 'print join ",", ("a".."zz")'
perl -le 'print join ",", "aa".."zz"'
```

#### 创建一个十六进制查找表

```
@hex = (0..9, "a".."f")
perl -le '
  $num = 255;
  @hex = (0..9, "a".."f");
  while ($num) {
    $s = $hex[($num % 16)].$s;
    $num = int $num/16;
  }
  print $s
'
perl -le 'printf("%x", 255)'
perl -le '$num = "ff"; print hex $num'
```

#### 生成一个随机的八字符密码

```
perl -le 'print map { ("a".."z")[rand 26] } 1..8'
perl -le 'print map { ("a".."z", 0..9)[rand 36] } 1..8'
```

#### 创建一个特定长度的字符串

```
perl -le 'print "a"x50'
perl -e 'print "a"x1024'
perl -le '@list = (1,2)x20; print "@list"'
```

#### 从字符串创建一个数组

```
@months = split ' ', "Jan Feb Mar Apr May Jun Jul Aug Sep Oct Nov Dec"
@months = qw/Jan Feb Mar Apr May Jun Jul Aug Sep Oct Nov Dec/
```

#### 从命令行参数创建一个字符串

```
perl -le 'print "(", (join ",", @ARGV), ")"' val1 val2 val3
perl -le '
  print "INSERT INTO table VALUES (", (join ",", @ARGV), ")"
' val1 val2 val3
```

#### 查找字符串中字符的数字值

```
perl -le 'print join ", ", map { ord } split //, "hello world"'
perl -le 'print join ", ", unpack("C*", "hello world")'
perl -le '
  print join ", ", map { sprintf "0x%x", ord $_ } split //, "hello world"
'
perl -le '
  print join ", ", map { sprintf "%o", ord $_ } split //, "hello world"
'
perl -le '
  print join ", ", map { sprintf "%#o", ord $_ } split //, "hello world"
'
```

#### 将一组数字 ASCII 值转换为字符串

```
perl -le '
  @ascii = (99, 111, 100, 105, 110, 103);
  print pack("C*", @ascii)
'
perl -le '
  @ascii = (99, 111, 100, 105, 110, 103);
  $str = join "", map chr, @ascii;
  print $str
'
perl -le 'print map chr, 99, 111, 100, 105, 110, 103'
perl -le 'print map chr, @ARGV' 99 111 100 105 110 103
```

#### 生成一个包含从 1 到 100 的奇数的数组

```
perl -le '@odd = grep {$_ % 2 == 1} 1..100; print "@odd"'
perl -le '@odd = grep { $_ & 1 } 1..100; print "@odd"'
```

#### 生成一个包含从 1 到 100 的偶数的数组

```
perl -le '@even = grep {$_ % 2 == 0} 1..100; print "@even"'
```

#### 查找字符串的长度

```
perl -le 'print length "one-liners are great"'
```

#### 查找数组中元素的数量

```
perl -le '@array = ("a".."z"); print scalar @array'
perl -le '@array = ("a".."z"); print $#array + 1'
perl -le 'print scalar @ARGV' *.txt
perl -le 'print scalar (@ARGV=<*.txt>)'
```

## C.5 文本转换与替换

#### 对字符串进行 ROT13 加密

```
perl -le '$str = "bananas"; $str =~ y/A-Za-z/N-ZA-Mn-za-m/; print $str'
perl -lpe 'y/A-Za-z/N-ZA-Mn-za-m/' file
perl -pi.bak -e 'y/A-Za-z/N-ZA-Mn-za-m/' file
```

#### 对字符串进行 Base64 编码

```
perl -MMIME::Base64 -e 'print encode_base64("string")'
perl -MMIME::Base64 -0777 -ne 'print encode_base64($_)' file
```

#### 对字符串进行 Base64 解码

```
perl -MMIME::Base64 -le 'print decode_base64("base64string")'
perl -MMIME::Base64 -0777 -ne 'print decode_base64($_)' file
```

#### 对字符串进行 URL 编码

```
perl -MURI::Escape -le 'print uri_escape("http://example.com")'
```

#### 对字符串进行 URL 解码

```
perl -MURI::Escape -le 'print uri_unescape("http%3A%2F%2Fexample.com")'
```

#### 对字符串进行 HTML 编码

```
perl -MHTML::Entities -le 'print encode_entities("<html>")'
```

#### 对字符串进行 HTML 解码

```
perl -MHTML::Entities -le 'print decode_entities("&lt;html&gt;")'
```

#### 将所有文本转换为大写

```
perl -nle 'print uc'
perl -ple '$_ = uc'
perl -nle 'print "\U$_"'
```

#### 将所有文本转换为小写

```
perl -nle 'print lc'
perl -nle 'print "\L$_"'
```

#### 仅将每行的首字母大写

```
perl -nle 'print ucfirst lc'
perl -nle 'print "\u\L$_"'
```

#### 反转字母大小写

```
perl -ple 'y/A-Za-z/a-zA-Z/'
```

#### 将每一行标题化

```
perl -ple 's/(\w+)/\u$1/g'
```

#### 去除每行开头的空白字符（空格、制表符）

```
perl -ple 's/^[ \t]+//'
perl -ple 's/^\s+//'
```

#### 去除每行结尾的空白字符（空格、制表符）

```
perl -ple 's/[ \t]+$//'
perl -ple 's/\s+$//'
```

#### 去除每行开头和结尾的空白字符（空格、制表符）

```
perl -ple 's/^[ \t]+|[ \t]+$//g'
perl -ple 's/^\s+|\s+$//g'
```

#### 将 UNIX 换行符转换为 DOS/Windows 换行符

```
perl -pe 's|\012|\015\012|'
```

#### 将 DOS/Windows 换行符转换为 UNIX 换行符

```
perl -pe 's|\015\012|\012|'
```

#### 将 UNIX 换行符转换为 Mac 换行符

```
perl -pe 's|\012|\015|'
```

#### 在每一行上将“foo”替换为“bar”

```
perl -pe 's/foo/bar/'
perl -pe 's/foo/bar/g'
```

#### 在匹配“baz”的行上，将“foo”替换为“bar”

```
perl -pe '/baz/ && s/foo/bar/'
perl -pe 's/foo/bar/ if /baz/'
```

#### 以倒序打印段落

```
perl -00 -e 'print reverse <>' file
```

#### 打印所有行的倒序

```
perl -lne 'print scalar reverse $_'
perl -lne 'print scalar reverse'
perl -lpe '$_ = reverse $_'
perl -lpe '$_ = reverse'
```

#### 以倒序打印列

```
perl -alne 'print "@{[reverse @F]}"'
perl -F: -alne 'print "@{[reverse @F]}"'
perl -F: -alne '$" = ":"; print "@{[reverse @F]}"'
```

## C.6 有选择地打印和删除行

#### 打印文件的第一行（模拟 head -1）

```
perl -ne 'print; exit' file
perl -i -ne 'print; exit' file
perl -i.bak -ne 'print; exit' file
```

#### 打印文件的前 10 行（模拟 head -10）

```
perl -ne 'print if $. <= 10' file
perl -ne '$. <= 10 && print' file
perl -ne 'print if 1..10' file
perl -ne 'print; exit if $. == 10' file
```

#### 打印文件的最后一行（模拟 tail -1）

```
perl -ne '$last = $_; END { print $last }' file
perl -ne 'print if eof' file
```

#### 打印文件的最后 10 行（模拟 tail -10）

```
perl -ne 'push @a, $_; @a = @a[@a-10..$#a] if @a>10; END { print @a }' file
perl -ne 'push @a, $_; shift @a if @a>10; END { print @a }' file
```

#### 仅打印匹配正则表达式的行

```
perl -ne '/regex/ && print'
perl -ne 'print if /regex/'
```

#### 仅打印不匹配正则表达式的行

```
perl -ne '!/regex/ && print'
perl -ne 'print if !/regex/'
perl -ne 'print unless /regex/'
perl -ne '/regex/ || print'
```

#### 打印匹配正则表达式之前的每一行

```
perl -ne '/regex/ && $last && print $last; $last = $_'
```

#### 打印匹配正则表达式之后的每一行

```
perl -ne 'if ($p) { print; $p = 0 } $p++ if /regex/'
perl -ne '$p && print && ($p = 0); $p++ if /regex/'
perl -ne '$p && print; $p = /regex/'
```

#### 打印匹配正则表达式 AAA 和 BBB 的所有行，顺序不限

```
perl -ne '/AAA/ && /BBB/ && print'
```

#### 打印不匹配正则表达式 AAA 和 BBB 的行

```
perl -ne '!/AAA/ && !/BBB/ && print'
```

#### 打印匹配正则表达式 AAA 后跟 BBB 后跟 CCC 的行

```
perl -ne '/AAA.*BBB.*CCC/ && print'
```

#### 打印至少 80 个字符长的行

```
perl -ne 'print if length >= 80'
perl -lne 'print if length >= 80'
```

#### 打印所有长度小于 80 个字符的行

```
perl -ne 'print if length() < 80'
```

#### 仅打印第 13 行

```
perl -ne '$. == 13 && print && exit'
```

#### 打印除第 27 行外的所有行

```
perl -ne '$. != 27 && print'
perl -ne 'print if $. != 27'
perl -ne 'print unless $. == 27'
```

#### 仅打印第 13 行、第 19 行和第 67 行

```
perl -ne 'print if $. == 13 || $. == 19 || $. == 67'
perl -ne '
  @lines = (13, 19, 88, 290, 999, 1400, 2000);
  print if grep { $_ == $. } @lines
'
```

#### 打印第 17 到 30 行的所有内容

```
perl -ne 'print if $. >= 17 && $. <= 30'
perl -ne 'print if 17..30'
```

#### 打印两个正则表达式之间的所有行（包括匹配的行）

```
perl -ne 'print if /regex1/../regex2/'
```

#### 打印最长的行

```
perl -ne '
  $l = $_ if length($_) > length($l);
  END { print $l }
'
```

#### 打印最短的行

```
perl -ne '
  $s = $_ if $. == 1;
  $s = $_ if length($_) < length($s);
  END { print $s }
'
```

#### 打印所有包含数字的行

```
perl -ne 'print if /\d/'
```

#### 打印所有仅包含数字的行

```
perl -ne 'print if /^\d+$/'
perl -lne 'print unless /\D/'
```

#### 打印所有仅包含字母的行

```
perl -ne 'print if /^[[:alpha:]]+$/
```

#### 打印每隔一行的内容

```
perl -ne 'print if $. % 2'
```

#### 打印每隔一行的内容，从第二行开始

```
perl -ne 'print if $. % 2 == 0'
perl -ne 'print unless $. % 2'
```

#### 仅打印所有重复的行一次

```
perl -ne 'print if ++$a{$_} == 2'
```

#### 打印所有唯一的行

```
perl -ne 'print unless $a{$_}++'
```

## C.7 有用的正则表达式

#### 匹配看起来像 IP 地址的内容

```
/^\d{1,3}\.\d{1,3}\.\d{1,3}\.\d{1,3}$/
/^(\d{1,3}\.){3}\d{1,3}$/
perl -ne 'print if /^\d{1,3}\.\d{1,3}\.\d{1,3}\.\d{1,3}$/'
```

#### 测试一个数字是否在 0 到 255 的范围内

```
/^([0-9]|[0-9][0-9]|1[0-9][0-9]|2[0-4][0-9]|25[0-5])$/
perl -le '
  map { $n++ if /^([0-9]|[0-9][0-9]|1[0-9][0-9]|2[0-4][0-9]|25[0-5])$/ } 0..255;
  END { print $n }
'
perl -le '
  map { $n++ if /^([0-9]|[0-9][0-9]|1[0-9][0-9]|2[0-4][0-9]|25[0-5])$/ } 0..1000;
  END { print $n }
'
```

#### 匹配 IP 地址

```
my $ip_part = qr/[0-9]|[0-9][0-9]|1[0-9][0-9]|2[0-4][0-9]|25[0-5]/;
if ($ip =~ /^$ip_part\.$ip_part\.$ip_part\.$ip_part$/) {
  print "valid ip\n";
}
if ($ip =~ /^($ip_part\.){3}$ip_part$/) {
  print "valid ip\n";
}
perl -ne '
  $ip_part = qr|([0-9]|[0-9][0-9]|1[0-9][0-9]|2[0-4][0-9]|25[0-5])|;
  print if /^($ip_part\.){3}$ip_part$/
'
```

#### 检查一个字符串是否像电子邮件地址

```
/\S+@\S+\.\S+/
use Email::Valid;
print Email::Valid->address('cats@catonmat.net') ? 'valid email' : 'invalid email';
perl -MEmail::Valid -ne 'print if Email::Valid->address($_)'
```

#### 检查一个字符串是否是数字

```
/^\d+$/
/^[+-]?\d+$/
/^[+-]?\d+\.?\d*$/
perl -MRegexp::Common -ne 'print if /$RE{num}{real}/'
perl -MRegexp::Common -ne 'print if /$RE{num}{hex}/'
perl -MRegexp::Common -ne 'print if /$RE{num}{oct}/'
perl -MRegexp::Common -ne 'print if /$RE{num}{bin}/'
```

#### 检查一个单词在字符串中是否出现两次

```
/(word).*\1/
```

#### 将字符串中的所有整数增加一

```
$str =~ s/(\d+)/$1+1/ge
perl -MRegexp::Common -pe 's/($RE{num}{real})/$1+1/ge'
```

#### 从 HTTP 头中提取 HTTP 用户代理字符串

```
/^User-Agent: (.+)$/
```

#### 匹配可打印的 ASCII 字符

```
/[ -~]/
```

#### 提取两个 HTML 标签之间的文本

```
m|<strong>([^<]*)</strong>|
m|<strong>(.*?)</strong>|
```

#### 将所有<b>标签替换为<strong>

```
$html =~ s|<(/)?b>|<$1strong>|g
```

#### 从正则表达式中提取所有匹配项

```
my @matches = $text =~ /regex/g;
```
