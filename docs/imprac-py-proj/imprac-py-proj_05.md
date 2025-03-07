## 第六章：**英国内战密码编码**

![image](img/common01.jpg)

1587 年，苏格兰的玛丽女王因一张纸条丧命。55 年后，查理一世的支持者约翰·特雷瓦尼恩爵士因一张纸条保住了性命。是什么造成了这种差异？隐写术。

*隐写术*是一种经得起时间考验的做法，用来将信息隐藏得如此精妙，以至于其存在根本不会被怀疑。这个名字源自希腊语，意为“隐藏的文字”，而一个非常字面意义上的希腊示例是用带有蜡层的木板写字，然后刮掉蜡层，在木板上写字，再覆盖一层新的光滑蜡膜。一个现代的例子是通过微妙地改变图像的颜色成分将信息嵌入图像中。即便是一张简单的 8 位 JPEG 图像，也包含了人眼无法察觉的更多颜色，因此在没有数字处理或过滤的情况下，信息几乎是不可见的。

在本章中，你将使用*无效密码*，这其实根本不是一种密码，而是一种将明文隐藏在其他非密码材料中的隐写技术。*Null*意味着“没有”，所以在无效密码中，你选择不加密信息。以下是使用每个单词的第一个字母的无效密码示例：

**N**ice **u**ncles **l**ive **l**onger. **C**ruel, **i**nsensitive **p**eople **h**ave **e**ternal **r**egrets.

首先，你将编写代码来找到救了约翰爵士的隐蔽讯息，然后你将完成一个更为困难的任务——编写一个无效密码。最后，你将有机会编写一个程序，如果玛丽使用了该程序的输出，可能会保住她的头颅。

### **项目 #10：特雷瓦尼恩密码**

玛丽女王依靠隐写术和加密技术来保护她的讯息。这个策略是有效的，但她的应用存在缺陷。她在不知情的情况下依赖一个名叫吉尔伯特·吉福德的双重间谍来走私她的讯息。吉福德首先将讯息交给伊丽莎白女王的间谍头目，后者破解了密码，并用伪造的讯息替换，诱使玛丽自陷囹圄。接下来，正如他们所说的，这就是历史。

对于约翰·特雷瓦尼恩来说，结局更加美好。约翰爵士是一位杰出的骑士，他在英国内战中帮助查理一世抵抗奥利弗·克伦威尔，后来被捕并关押在科尔切斯特城堡。就在他被处决前一天，他收到了来自一位朋友的信。信件并非走私，而是直接交到看守手中，尽管他们检查过信件，但并没有发现任何欺骗。读完信后，约翰爵士请求独自到教堂祈祷。当他的看守回来找他时，他已经消失了。

这是约翰爵士收到的讯息：

尊敬的约翰爵士：希望，这是受苦者最好的安慰，我恐怕现在无法为您提供太多帮助。我想对您说的仅有这一点：如果我能偿还我欠您的，我会毫不犹豫地为您效劳。虽然我能做的不多，但我能做的，您可以非常确信我一定会做。我知道，如果死亡来临，普通人会害怕，但它不会让您害怕，您把它视为对忠诚的至高荣誉的奖赏。祈祷您能避免这杯苦酒。我不担心您会抱怨任何苦难；只有在通过顺从您能避免它们时，那才是智者的行为。告诉我，如果您能的话，我可以为您做任何您希望我做的事情。将军将在星期三回去。恭敬的，您的仆人，R.T.

正如您可能猜到的，这封看似无害的信中包含了一个隐藏的信息，以下是用粗体显示的部分：

尊敬的约翰爵士：希望，这是受苦者最好的安慰，我恐怕现在无法为您提供太多帮助。我想对您说的仅有这一点：如果我能偿还我欠您的，我会毫不犹豫地为您效劳。虽然我能做的不多，但我能做的，您可以非常确信我一定会做。我知道，如果死亡来临，普通人会害怕，但它不会让您害怕，您把它视为对忠诚的至高荣誉的奖赏。祈祷您能避免这杯苦酒。我不担心您会抱怨任何苦难；只有在通过顺从您能避免它们时，那才是智者的行为。告诉我，如果您能的话，我可以为您做任何您希望我做的事情。将军将在星期三回去。恭敬的，您的仆人，R.T.

这个空白密码利用标点符号后的每三个字母来让约翰爵士知道“教堂东端的面板滑动”。有传闻说，后来在城堡的一堵墙的壁龛中发现了一个狭窄楼梯的遗迹。这个通道在发现时被封锁，但它可能是约翰爵士在 1642 年左右的逃生通道。

这种临时逃生如果使用传统的密码是无法实现的。只有通过巧妙地隐藏消息，使用隐写术，作者才能如此迅速地将其交到约翰爵士手中。而空白密码的美妙之处在于，即使约翰爵士不知道模式，但怀疑消息存在，他也能相当迅速地找到它。

如果约翰爵士的朋友更小心一点，隐藏的是加密的密文而非明文，约翰爵士可能就无法在剩余的短时间内解密出消息——除非他事先得知了密码的类型和密钥。

**目标**

编写代码，找出空白密码中标点符号后隐藏的字母，并让用户选择在标点符号后查找的字母数量，以寻找解决方案。

#### ***策略与伪代码***

空白密码依赖于发送者和接收者都知道的重复模式。例如，每三个单词可能是实际消息的一部分，或者更好的是，每三个单词的最后一个字母。在 Trevanion 密码中，它是标点符号后的第三个字母。

要找到 Trevanion 密码，假设标点符号是开始计数的信号，然后编写代码找到每个标点符号后的*第 n*个字母，并将这些字母保存到字符串或列表中。一旦你弄清楚如何做，你就可以轻松编辑代码，以便它适用于任何起始点，例如每个大写字母单词、每个单词的第二个字母，或者每三个单词的起始字母。

唯一的争议点涉及标点符号。例如，空白密码的作者*是否希望*将标点符号包括在明文中？如何处理计数范围内出现的第二个标点符号？如果两个标点符号连续出现会怎样？

如果仔细观察 Trevanion 密码，你应该会看到由于重复使用词语*’tis*，导致了双重标点符号。消息结尾处还有一串标点符号，作者在此提供了自己的首字母。为了解决这个问题，约翰爵士和他的朋友可能在约翰爵士入狱之前就已经制定了一些规则，或者约翰爵士通过反复试验得出了这些规则。

根据消息结尾，标点符号不包括在字母计数内。如果约翰爵士的朋友打算将它们包括在内，那么隐藏消息将以大写的*T*结尾，因为*T*在标点符号后面三*个字符*，而关键是这不是三个*字母*。这意味着，如果读者在计数范围内遇到标点符号，他们必须重新开始计数。

所以这些是规则：

+   每遇到一个标点符号，启动字母计数。

+   如果遇到标点符号，则重置计数。

+   标点符号不能成为明文消息的一部分。

由于你可能不知道字母计数应该是多少，因此编写代码以检查所有计数，直到用户提供的限制。伪代码相当简单：

```py
Load a text file and strip it of whitespace
Get user input on how many letters after punctuation to look ahead and examine
Loop through number of letters from 1 to this lookahead value
    Start an empty string to hold the translation
    Start a counter
    Start a ➊first-found marker and set to False
    Loop through characters in the text
        If character is punctuation
            Counter = 0
            First-found = True
        Otherwise, if ➋first-found is True
            Counter + 1
        If counter = lookahead value
            Add character to translation string
    Display translation for this lookahead value
```

请注意，第一次找到的变量➊将保持`False`，直到遇到标点符号为止，此时它将设置为`True` ➋。这防止程序在找到第一个标点符号之前进行计数。

现在你准备好编写代码了！

#### ***Trevanion 密码代码***

本节中的代码将找到使用特定字母数编码的 Trevanion 类型空白密码，该字母数位于每个标点符号后。你还需要包含 Trevanion 密码的文本文件。你可以从*[`www.nostarch.com/impracticalpython/`](https://www.nostarch.com/impracticalpython/)*下载脚本和文本文件，分别命名为*null_cipher_finder.py*和*trevanion.txt*。请将这些文件保存在同一文件夹中。

##### **加载文本**

Listing 5-1 导入一些有用的模块并加载包含空密码的文本文件。

*null_cipher_finder.py,* 第一部分

```py
➊ import sys
   import string

➋ def load_text(file):
       """Load a text file as a string."""
    ➌ with open(file) as f:
        ➍ return f.read().strip()
```

*Listing 5-1: 导入模块并加载空密码文本*

首先，导入现在已经熟悉的 `sys` 模块，以便处理用户输入过程中可能发生的异常 ➊。还需要导入 `string` 模块，以便访问一些有用的常量集合，比如字母和标点符号。

接下来，定义一个函数来加载包含空密码的文本文件 ➋。这个函数类似于你在 第二章 中用来加载字典文件的函数。稍后 `main()` 函数会调用它来实际加载文件。

使用 `with` 打开文件 ➌ 来启动 `load_text()` 函数。通过使用 `with`，你可以确保文件在加载后会自动关闭。使用 `read()` 加载文件内容，并使用 `strip()` 去除前后的空白字符。注意，你可以在同一行中结合 `return` 语句来完成这一步 ➍。

##### **寻找隐藏的信息**

Listing 5-2 定义了一个查找隐藏信息的函数。它接受两个参数，第一个是消息，即去除空白字符后的原始文本文件，第二个是标点符号后要检查的字母数。这个检查值是从用户那里获得的，作为 `main()` 函数的一部分。

*null_cipher_finder.py,* 第二部分

```py
def solve_null_cipher(message, lookahead):
    """Solve a null cipher based on number of letters after punctuation mark.

    message = null cipher text as string stripped of whitespace
    lookahead = endpoint of range of letters after punctuation mark to examine
    """
 ➊ for i in range(1, lookahead + 1):
        ➋ plaintext = ''
           count = 0
           found_first = False
        ➌ for char in message:
            ➍ if char in string.punctuation:
                   count = 0
                   found_first = True
            ➎ elif found_first is True:
                   count += 1
            ➏ if count == i:
                   plaintext += char
        ➐ print("Using offset of {} after punctuation = {}".
                 format(i, plaintext))
           print()
```

*Listing 5-2: 搜索隐藏的字母*

将 `lookahead` 值视为 `for` 循环中的范围终点，这样你就可以检查消息中所有介于其间的字母，查找是否存在隐藏的信息。将范围设置为 `(1, lookahead + 1)` ➊；这样，你就可以从标点符号后的第一个字母开始，并将用户选择的字母包含在评估中。

现在，分配几个变量 ➋。首先，初始化一个空字符串用于存储翻译后的明文。然后，将计数器设置为 `0`。最后，将 `found_first` 变量设置为 `False`。记住，程序使用这个变量来推迟计数，直到遇到第一个标点符号。

接下来，开始遍历消息中的字符 ➌。如果遇到标点符号，重置计数器为 `0`，并将 `found_first` 设置为 `True` ➍。如果已经找到了标点符号，而当前字符不是标点符号，则将计数器加 1 ➎。如果找到了你要找的字母—即计数器已经达到当前的 `lookahead` 值（`i`）—则将字母添加到明文字符串中 ➏。

当你检查完消息中所有字符以匹配当前的 `lookahead` 值后，显示当前的密钥和翻译 ➐。

##### **定义 main() 函数**

Listing 5-3 定义了 `main()` 函数。你可能记得在 第三章 中提到过，`main()` 函数就像程序的项目经理：它接收输入，跟踪进度，并告诉其他函数何时工作。

*null_cipher_finder.py,* 第三部分

```py
def main():
    """Load text, solve null cipher."""
    # load & process message:
 ➊ filename = input("\nEnter full filename for message to translate: ")
 ➋ try:
        loaded_message = load_text(filename)
    except IOError as e:
        print("{}. Terminating program.".format(e), file=sys.stderr)
        sys.exit(1)
 ➌ print("\nORIGINAL MESSAGE =")
    print("{}".format(loaded_message), "\n")
    print("\nList of punctuation marks to check = {}".
          format(string.punctuation), "\n")   

    # remove whitespace:
 ➍ message = ''.join(loaded_message.split())

    # get range of possible cipher keys from user:
 ➎ while True:
     ➏ lookahead = input("\nNumber of letters to check after " \
                          "punctuation mark: ")
     ➐ if lookahead.isdigit():
            lookahead = int(lookahead)
            break
        else:
         ➑ print("Please input a number.", file=sys.stderr)
    print()

    # run function to decode cipher
 ➒ solve_null_cipher(message, lookahead)
```

*列表 5-3：定义了* main() *函数*

首先询问用户文件的名称（名称 + 扩展名） ➊，然后使用 `try` 调用 `load_text()` 函数 ➋。如果找不到文件，打印红色错误信息——对于使用 IDLE 窗口的用户——并使用 `sys.exit(1)` 退出程序，其中 `1` 表示错误终止。

打印消息后，列出 `string` 模块中的标点符号 ➌。程序将只识别这个列表中的字符作为标点符号。

接下来，取出已加载的消息并移除所有空格 ➍。你只会计算字母和标点符号，因此空格会妨碍操作。启动一个 `while` 循环，在用户输入无效值时继续请求输入 ➎。询问用户要检查标点符号后几个字母 ➏。这将作为一个范围，起始值为 1，结束值为用户选择的数字加 1。如果输入值是数字 ➐，将其转为整数，因为 `input` 返回的是字符串。然后，使用 `break` 退出循环。

如果用户输入无效值，如“Bob”，则使用 `print` 语句请求一个数字，并且对于 shell 用户，使用 `sys.stderr` 使字体变为红色 ➑。然后，`while` 循环会重复请求输入。

将 `lookahead` 变量与 `message` 一起传递给 `solve_null_cipher` 函数 ➒。现在只剩下调用 `main()` 函数。

##### **运行 main() 函数**

在全局空间中，通过调用 `main()` 完成代码——但仅当程序以独立模式运行，而不是被导入到另一个程序中时（列表 5-4）。

*null_cipher_finder.py,* 第四部分

```py
if __name__ == '__main__':
    main()
```

*列表 5-4：调用了* main() *函数*

以下是使用 Trevanion 密码作为输入的完成程序的示例输出：

```py
Enter full filename for message to translate: trevanion.txt

ORIGINAL MESSAGE =
Worthie Sir John: Hope, that is the beste comfort of the afflicted, cannot
much, I fear me, help you now. That I would saye to you, is this only: if ever
I may be able to requite that I do owe you, stand not upon asking me. 'Tis not
much I can do: but what I can do, bee you verie sure I wille. I knowe that,
if deathe comes, if ordinary men fear it, it frights not you, accounting for
it for a high honour, to have such a rewarde of your loyalty. Pray yet that
you may be spared this soe bitter, cup. I fear not that you will grudge any
sufferings; onlie if bie submission you can turn them away, 'tis the part of a
wise man. Tell me, an if you can, to do for you anythinge that you wolde have
done. The general goes back on Wednesday. Restinge your servant to command.
R.T.

List of punctuation marks to check = !"#$%&'()*+,-./:;<=>?@[\]^_`{|}~

Number of letters to check after punctuation mark: 4

Using offset of 1 after punctuation = HtcIhTiisTbbIiiiatPcIotTatTRRT

Using offset of 2 after punctuation = ohafehsftiuekfftcorufnienohe

Using offset of 3 after punctuation = panelateastendofchapelslides

Using offset of 4 after punctuation = etnapthvnnwyoerroayaitlfogt
```

在此输出中，程序已检查到标点符号后的第四个字母，但正如您所看到的，它是使用标点符号后面的三个字母找到解决方案的。

### **项目 #11：编写空密码**

这是一个基于每个单词开头的非常弱的空密码的未完成示例。花一点时间尝试完成这个句子：

H__________ e__________ l__________ p__________ m__________ e__________.

你可能觉得很困难，因为无论是使用字母还是完整单词，要产生一个不显得生硬且不引起怀疑的空密码都需要艰苦的工作和时间。问题的核心是上下文。如果密码包含在通信中，那么这些通信必须连贯，以避免引起怀疑。这意味着它必须涉及一个相关话题，并且在合理的句子数范围内保持与该话题的一致性。正如你可能看到的，草拟关于任何话题的一个句子都不是一件容易的事！

关键是要可靠地避免上下文，而一个很好的方法就是使用列表。没有人期望购物清单是严格有序的或者合乎逻辑的。列表还可以根据接收者量身定制。例如，通讯员可能会讨论书籍或电影，并交换他们最喜欢的书单或影单。囚犯可能会开始学习外语，并从导师那里定期接收词汇表。商人可能会从其中一个仓库收到每月的库存清单。通过使用列表，即使单词被打乱，仍然可以尊重上下文，以便正确的字母出现在正确的位置。

**目标**

编写代码，将空白密码隐藏在一个单词列表中。

#### ***列表密码代码***

*list_cipher.py* 代码，在 清单 5-5 中，将一个空白密码嵌入在一个字典单词的列表中，伪装成词汇训练。你还需要在 第二章 和 第三章 中使用过的 *load_dictionary.py* 程序。你可以从 *[`www.nostarch.com/impracticalpython/`](https://www.nostarch.com/impracticalpython/)* 下载这个文件以及以下脚本。最后，你还需要在 第二章 和 第三章 中使用过的字典文件。你可以在 第 2-1 表 中找到适合的在线字典，位置在 第 20 页。所有上述文件应该保存在同一个文件夹中。

*list_cipher.py*

```py
➊ from random import randint
   import string
   import load_dictionary

   # write a short message that doesn't contain punctuation or numbers!
   input_message = "Panel at east end of chapel slides"

   message = ''
   for char in input_message:
    ➋ if char in string.ascii_letters:
           message += char
   print(message, "\n")
➌ message = "".join(message.split())

➍ # open dictionary file
   word_list = load_dictionary.load('2of4brif.txt')

   # build vocabulary word list with hidden message
➎ vocab_list = []
➏ for letter in message:
       size = randint(6, 10)
    ➐ for word in word_list:
           if len(word) == size and word[2].lower() == letter.lower()\
           and word not in vocab_list:
               vocab_list.append(word)
               break

➑ if len(vocab_list) < len(message):
       print("Word List is too small. Try larger dictionary or shorter message!")
   else:
       print("Vocabulary words for Unit 1: \n", *vocab_list, sep="\n")
```

*清单 5-5：在列表中隐藏空白密码*

首先，导入 `random` 模块的 `randint()` 函数 ➊。这允许选择一个（伪）随机的整数值。然后加载 `string` 模块，以访问 ASCII 字母。最后，导入你的 `load_dictionary` 模块。

接下来，编写一条简短的秘密消息。请注意，相关的注释禁止使用标点符号或数字。尝试在字典文件的内容中使用这些会引发问题。因此，通过检查 `string.ascii_letters` 中的成员来筛选出除了字母以外的所有内容，`string.ascii_letters` 包含大写字母和小写字母 ➋：

```py
'abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ'
```

显示消息后，去除空白字符 ➌。加载字典文件 ➍，并开始一个空列表来保存词汇单词 ➎。

使用 `for` 循环遍历消息中的每个字母 ➏。命名一个 `size` 变量，并使用 `randint()` 函数为其分配一个介于 6 到 10 之间的随机值。这个变量将确保单词足够长，看起来像是一个可信的词汇。你可以根据需要将最大值设置得更高。

嵌套另一个`for`循环，使用它来遍历字典中的单词 ➐，检查它们的长度是否与`size`变量相符，并将第 2 个索引位置（即单词的第三个字母）的小写字母与消息循环中的当前（小写）字母进行比较。你可以更改单词中的索引值，但要确保它不超过最低可能的`size`变量减去 1！最后的比较防止同一个单词被使用两次。如果单词通过测试，则将其附加到`vocab_list`中，并继续处理消息中的下一个字母。

一个典型的字典文件应包含足够的单词来加密一个简短的消息。但是，为了安全起见，使用条件语句检查`vocab_list`的长度是否不小于消息的长度 ➑。如果较短，那么你在到达消息末尾之前就已经用完了单词，应该给用户打印警告。否则，打印单词列表。

#### ***列表密码输出***

下面是代码的输出（我已将每第三个字母突出显示，以便阅读，尽管没有任何辅助工具消息也很容易找到）：

```py
Panelateastendofchapelslides

Vocabulary words for Unit 1:

alphabets
abandoning
annals
aberration
ablaze
abandoned
acting
abetted
abasement
abseil
activated
adequately
abnormal
abdomen
abolish
affecting
acceding
abhors
abalone
ampersands
acetylene
allegation
absconds
aileron
acidifying
abdicating
adepts
absent
```

使用字符宽度一致的字体并堆叠单词会严重削弱密码的安全性。我们将在《拯救玛丽》一章中，第 102 页讨论如何解决这个问题。

### **总结**

在本章中，你编写了一个程序，揭示了特雷瓦尼翁型空密码中的隐藏信息。然后，你又编写了第二个程序，生成了一个空密码，并将其隐藏在语言学习者的词汇表中。在接下来的实践项目中，你可以探索如何使这个列表密码更加安全。

### **进一步阅读**

关于玛丽·斯图亚特女王和约翰·特雷瓦尼翁爵士的更多详情可以在*《神秘信息：密码和暗号的历史》*（企鹅集团，2009 年）和*《密码书：从古埃及到量子密码学的秘密科学》*（Anchor，2000 年）中找到，前者作者为加里·布莱克伍德，后者为西蒙·辛格。

### **实践项目**

现在你已经是一个空密码的专家，试试看能否改变玛丽·斯图亚特女王的命运，然后偷看一下约翰爵士最机密的信件。

#### ***拯救玛丽***

编码的最佳部分是思考问题及如何解决它们。让我们回顾一下玛丽·斯图亚特女王的悲剧案例。我们知道的情况如下：

+   玛丽被禁止通信，因此信件必须通过走私方式送入。这意味着叛徒吉尔伯特·吉福德无法从中除去。吉福德是玛丽唯一认识的、能送信的人。

+   玛丽和她的通信者对不安全的密码过于信任，因此言语过于轻率。如果他们少一点信心，也许会更加忍耐。

+   玛丽的狱卒们发现了一个明显的密码本，他们认为里面包含了有罪证据，便继续工作，直到找到了它。

伪双重间谍吉福德并未掌握玛丽所使用密码的细节。现在，假设玛丽使用了空白密码。如果信件的内容有些煽动性——但并非叛国——那么她的监禁者可能会忽视这条消息。如果进行过粗略检查，使用可变模式可能足以阻止密码分析师。

正如你所看到的，将空白密码隐藏在列表中比隐藏在信件中要容易得多。支持玛丽的家庭列表可以用作此目的。这些家庭可以是已知的支持者，或者在马基雅维利式的扭曲中，既有朋友*也*有敌人！这条消息不会公开煽动叛乱，但足够接近，以至于缺乏加密会暗示根本没有使用任何形式的加密。

对于这个练习项目，编写一个程序，将消息“Give your word and we rise”嵌入一个姓氏列表中。为了隐藏消息中的字母，从第二个名字的第二个字母开始，接着是第三个名字的第三个字母，然后在剩余的单词中交替使用第二个和第三个字母。

除了未使用的名字外，在列表的前面包含“Stuart”和“Jacob”作为空白词，以帮助隐藏密码的存在。不要在这些空白名字中嵌入密码字母，并且在为下一个单词选择密码字母位置时，完全忽略它们；如果第二个字母在空白名字*之前*的单词中已使用，则在空白名字*之后*的单词中使用第三个字母。空白密码将占据以下加粗字母的位置（空白词的位置由你决定，但不要让它们影响模式）：

```py
First Second Third STUART Fourth Fifth JACOB Sixth Seventh Eighth
```

程序可以垂直或水平打印列表。姓名列表应该用一条简短的消息进行可信的介绍，但这条消息不应该是密码的一部分。

姓名列表可以从 *[`www.nostarch.com/impracticalpython/`](https://www.nostarch.com/impracticalpython/)* 下载，作为 *supporters.txt* 并加载为标准字典文件。你可以在附录和网上找到解决方案，文件名为 *save_Mary_practice.py*。

#### ***科尔切斯特捕捉***

你不再是那个喝醉的傻瓜，*你*在接下来的信件到达科尔切斯特城堡时，负责看守囚犯约翰·特雷瓦尼昂：

约翰爵士：你们这些人真是奇怪又太难了。不过，我们会团结起来，像你一样坚持到底。还有谁能爱敌人，失败时坚持不懈，仇恨并绝望？在我们都能做的情况下，让我们感受希望。- R.T.

这句话看起来很笨拙，即使对于 17 世纪来说也是如此，你决定在将其交给你的囚犯之前，仔细检查一下。

编写一个 Python 程序，输入 *n*，并根据每个第 *n* 个单词的第 *n* 个字母后的字母检查并显示空白密码。例如，输入 `2` 将会在这条消息中找到加粗字母：

所以，冷**茶**没能让**老**女人感到满意。

你可以从 *[`nostarch.com/impracticalpython/`](https://nostarch.com/impracticalpython/)* 下载消息的文本文件，文件名为 *colchester_message.txt*。解决方案可以在附录和在线找到，文件名为 *colchester_practice.py*。请将文本和 Python 文件保存在同一个文件夹中。
