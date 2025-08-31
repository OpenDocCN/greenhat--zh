## 第十二章：B

**使用 BITCOINJ 进行比特币编程**

在附录 A 中，我们尝试了 JavaScript 代码，通过自动化现有的钱包来执行一些基本的比特币钱包操作。在本附录中，我们将编写更强大的程序，直接将自己插入到比特币网络中。正如之前所讨论的那样，以这种方式编写的比特币程序——也就是说，那些不依赖于外部公司 API、避免依赖独立钱包程序的程序——通常是进行严肃比特币开发项目的最佳解决方案。

### 连接到比特币网络的最佳编程语言

如果你是一个经验丰富的程序员，很可能你有一个偏爱的编程语言来编写比特币程序。然而，目前只有少数成熟的库可以直接连接到比特币网络。其中一个是 C++ 参考实现，所有比特币系统的第一个完整客户端都是用它编写的。^(1) 有意支持比特币的大型企业公司应使用此库。然而，C++ 对初学者来说相对较难。

另一个成熟的库是用 Java 编写的，名为 bitcoinJ。^(2) 由于 Java 比 C++ 更易于使用，因此我们将使用这个库。（bitcoinJ 库也可以很容易地通过构建在 Java 虚拟机上的其他语言（如 Scala 或 Clojure）使用。）

目前，其他语言的选项相当有限。通过搜索互联网，你*会*找到适用于其他语言的比特币库，如 Ruby、C# 和 Python。然而，这些库大多数都处于极其早期的开发阶段，或者仅仅使用更为有限的 JSON-RPC 接口，这在附录 A 中有讨论。

**注意**

*目前正在进行一些严肃的尝试，旨在为 Go 编程语言提供完整的比特币客户端支持。然而，在撰写本文时，这些库要么仍处于早期开发阶段（例如 gocoin^(3))，要么缺少主要功能（例如 btcd ^(4))。*

无论你选择哪个库，请记住，必须保持警惕以确保安全。实际上，当你使用这些库时，你相当于将比特币王国的钥匙交给了库的作者。正如我们之前所警告的，技术上来说，将恶意代码引入这些库以窃取所有资金是相对容易的！

### 安装 Java、Maven 和 BitcoinJ 库

让我们为你的计算机准备好进行 Java 和 bitcoinJ 编程。以下步骤应适用于任何主要操作系统：Windows、Mac 或 Linux。

#### *步骤 1：安装 Java*

Java 编程语言由 Oracle 公司维护。你的电脑可能已经安装了 Java，但如果你从未进行过 Java 开发，电脑上可能只包含 Java JRE（Java 运行时环境），它能够运行 Java 程序，但不足以进行开发。你需要的是 Java JDK（Java 开发工具包）。要下载 JDK，可以在 Google 上搜索 *Java JDK*。第一个链接（应该是 Oracle 官方网站的链接）会把你引导到正确的下载页面。

**注意**

***Linux 开发者：**某些版本的 Linux 可能默认安装 OpenJDK 版本的 Java。在撰写本文时，这种非官方的 Java 变体缺少一些本教程所依赖的组件。因此，请在线查找有关在你的 Linux 版本上安装 Oracle JDK 的信息，并将其设置为系统默认的 Java 版本。*

#### *步骤 2：安装 Maven*

Maven 是一个用于 Java 的打包工具。基本上，你指定程序需要哪些库，Maven 会自动从互联网上下载它们并使其可供你的程序使用。这类似于在附录 A 中使用的 Node 包管理器。

你可以在 *[`maven.apache.org/`](https://maven.apache.org/)* 下载 Maven 并找到各操作系统的安装说明。只需按照 Maven 官方网站上的说明进行安装，或者在 Google 上搜索教程。由于已有超过一百万人遇到并解决了安装 Maven 的问题，如果你遇到安装问题，只需将问题输入 Google，极有可能会找到有用的解决方案。

**注意**

***Windows 用户：**在撰写本文时，关于 Windows 安装 Maven 的详细说明巧妙地隐藏在* [`maven.apache.org/download.cgi`](http://maven.apache.org/download.cgi)的最底部。 ***Mac/Linux 用户：**你可以在此使用你的包管理器：Mac 使用 brew install maven，Debian Linux 使用 sudo apt-get install maven。*

为确保 Maven 正确安装，在控制台中输入 `mvn --version` 时应该会显示版本信息。

对于典型的 Java 编程，我们可以到此为止，因为任何额外需要的库都可以通过 Maven 包系统下载。然而，由于我们正在编写与金钱相关的程序，bitcoinJ 的维护者要求所有开发者采取一些额外的预防措施，并安装几个直接与安全问题相关的工具。

#### *步骤 3：安装 Git*

出于安全原因，我们将安装 Git，这是一款流行的源代码管理工具。它还提供了从在线代码库安全下载源代码的功能，我们将依赖这些功能。可以从 *[`git-scm.com/`](http://git-scm.com/)* 下载 Git。

**注意**

*再一次，Mac 和 Linux 用户可以使用他们的包管理器：分别使用 brew install git 和 apt-get install git。你看到了一个规律吗？*

#### *步骤 4：安装 BitcoinJ*

BitcoinJ 是一个比特币库，能够“理解比特币”，并且可以直接连接到比特币网络。通过调用该库中的函数，我们可以实时发送和接收比特币。

我们将直接从其权威源构建 bitcoinJ。通过控制台导航到你希望安装 bitcoinJ 库的目录（你的*Home*目录是一个不错的选择）。

**注意**

*将会创建一个新的子目录，因此该目录中的其他现有文件不会受到此安装的影响。*

在控制台中输入以下内容（在 Windows 上，你可能需要使用安装 Git 时提供的特殊 Git Bash 控制台）：

```
> git clone https://github.com/bitcoinj/bitcoinj.git ➊
> cd bitcoinj
> git checkout cbbb1a2 ➋
> mvn install ➌

```

第一行从权威网站 ➊ 下载 bitcoinJ 代码。然后，我们使用 git 切换到该库的一个旧版本，使用`git checkout`命令 ➋。在本教程中，该命令将帮助我们避免因库的更新版本而可能出现的任何问题。然而，当你完成教程后，你可以切换到 bitcoinJ 的最新版本（`git checkout master`）来尝试其新特性。最后一行将该包安装到本地 Maven 包仓库 ➌。当我们从程序中引用 bitcoinJ 时，Maven 将从该仓库中获取，而不是从互联网上获取，这样我们可以确保使用的是未被篡改的库版本。

现在我们终于可以开始编程了！

### 为 hello-money 创建一个启动项目

在附录 A 中，我们创建了一个名为`Hello Money!`的 JavaScript 程序，用于检测何时向比特币地址发送资金。现在，我们将使用 Java 和 bitcoinJ 编写一个更复杂的程序，完成相同的任务。从控制台中，导航到你希望新程序所在的目录，比如你的计算机的*Home*或*Documents*文件夹。该目录下将会创建一个子目录。

现在输入以下命令让 Maven 创建一个空的启动项目：

```
mvn archetype:generate -DgroupId=hellomoney -DartifactId=hello-money
   -DarchetypeArtifactId=maven-archetype-quickstart -DinteractiveMode=false

```

这些命令将在当前位置创建一个名为*hello-money*的目录，并放置`hello-money`启动项目。

**注意**

*如果 Maven 在此过程中提示你输入答案，只需不断按 ENTER 键以使用默认设置。*

接下来，我们需要告诉 Maven 这个程序将使用一些外部库。我们通过编辑*pom.xml*文件来实现这一点，该文件现在应该已经存在于新目录中。在这个文件中，还应该有一个名为`<dependencies>`的部分，我们将在其中添加 bitcoinJ 作为新的依赖项。在之前的依赖项之后（即在`</dependency>`这一行*之后*），添加以下内容：

```
<dependency>
  <groupId>com.google</groupId>
  <artifactId>bitcoinj</artifactId>
  <version>0.8</version>
  <scope>compile</scope>
</dependency>

```

现在我们将向程序中添加一个名为`exec-maven-plugin`的插件。*插件*是一种特殊类型的库。`exec-maven-plugin`将使我们能够更轻松地从命令行运行已完成的程序。

在*pom.xml*文件的最底部（即在`</dependencies>`这一行*之后*），添加以下内容：

```
<build>
  <plugins>
    <plugin>
      <groupId>org.codehaus.mojo</groupId>
      <artifactId>exec-maven-plugin</artifactId>
      <version>1.2.1</version>
      <configuration>
        <arguments>
        </arguments>
        <mainClass>hellomoney.App</mainClass>
      </configuration>
    </plugin>
  </plugins>
</build>

```

现在我们准备好第一次运行这个*空的*程序作为测试了。要做到这一点，请在程序的目录中从控制台执行以下几行：

```
> mvn install ➊
> mvn exec:java ➋

Hello World!

```

第一行 ➊ 将所有必要的库加载到项目中，并将程序构建成 Java 字节码文件。第二行 ➋ 实际上运行程序。

如果程序成功运行，屏幕上应该会显示`Hello World!`。这意味着 Maven 已经成功创建了一个功能正常的 Java 程序，我们现在可以开始编写核心的比特币代码。

### 编写 hello-money 代码

我们的`hello-money`程序的完整代码如下。要将其添加到项目中，请打开文件*src/main/java/hellomoney/App.java*并将其内容替换为该程序：

```
package hellomoney;
import com.google.bitcoin.core.*;
import com.google.bitcoin.store.*;
import com.google.bitcoin.discovery.DnsDiscovery;
import java.io.File;
import java.math.BigInteger;

public class App
{
    public static void main( String[] args ) throws BlockStoreException
    {
        NetworkParameters params = NetworkParameters.prodNet();
        Wallet wallet = new Wallet(params);
        ECKey key = new ECKey();
        System.out.println("Public address: " +
            key.toAddress(params).toString());
        System.out.println("Private key: " +
            key.getPrivateKeyEncoded(params).toString());
        wallet.addKey(key);
        File file = new File("my-blockchain");
        SPVBlockStore store=new SPVBlockStore(params, file);
        BlockChain chain = new BlockChain(params, wallet, store);
        PeerGroup peerGroup = new PeerGroup(params, chain);
        peerGroup.addPeerDiscovery(new DnsDiscovery(params));
        peerGroup.addWallet(wallet);
        peerGroup.start();
        peerGroup.downloadBlockChain();
        wallet.addEventListener(new AbstractWalletEventListener()
            {
                public void onCoinsReceived(Wallet wallet,
                      Transaction tx, BigInteger prevBalance,
                      BigInteger newBalance)
                {
                    System.out.println("Hello Money! Balance: "
                        + newBalance + " satoshis");
                }
            });
         while(true){}
    }
}

```

接下来，运行命令`mvn install`，该命令会检查新程序代码的语法并将其构建成程序文件。如果构建成功，应该会显示`BUILD SUCCESS`的消息（以及大量其他复杂的消息）。

在运行程序之前，让我们一步一步地了解它是如何工作的。

#### *程序顶部的声明*

程序的第一行声明了包的名称：

```
package hellomoney;

```

接下来，我们声明程序将引用的所有库：

```
import com.google.bitcoin.core.*;
import com.google.bitcoin.store.*;
import com.google.bitcoin.discovery.DnsDiscovery;
import java.io.File;
import java.math.BigInteger;

```

其中三项引用是比特币相关的类：首先，我们将使用核心库来访问基本的比特币类（例如钱包和密钥的类）。其次，我们需要用于存储区块链的类（在 BitcoinJ 术语中称为*区块存储*）。第三，我们需要使用`DnsDiscovery`类，它帮助我们找到参与比特币网络的其他节点。我们导入`java.io.File`类，因为我们将把区块存储写入文件，并且导入`java.math.BigInteger`类来处理大整数。

现在让我们定义一个 Java 类来保存这个程序：

```
public class App
{
    public static void main( String[] args ) throws BlockStoreException
    {

```

程序代码存储在一个名为`App`的新类中，该类包含一个名为`main`的成员函数。我们在*pom.xml*文件中提到过这个`hellomoney.App`类，将其声明为程序的*主类*。

让我们逐行看看主函数中的代码。

#### *初始化我们的 Java 对象*

这是从 bitcoinJ 库中初始化我们需要的 Java 对象的代码。

```
NetworkParameters params = NetworkParameters.prodNet();➊
Wallet wallet = new Wallet(params);➋
ECKey key = new ECKey();➌
System.out.println("Public address: " +➍
    key.toAddress(params).toString());
System.out.println("Private key: " +➎
    key.getPrivateKeyEncoded(params).toString());
wallet.addKey(key);➏

```

我们首先获取主生产比特币网络的网络参数➊。虽然实际用于交易的只有一个真正的比特币网络，但使用真实货币彻底测试比特币系统是困难的；因此，比特币开发者还维护了一个名为*TestNet*的第二个比特币网络，仅供测试使用。`NetworkParameters`结构包含有关创世区块（区块链中的第一个区块）以及最大硬币数量等多个其他细节的信息，这些信息在主比特币网络和 TestNet 之间可能有所不同。通过将所有这些信息打包在`NetworkParameters`数据结构中，我们可以轻松地将程序连接到除主比特币网络之外的其他网络，如 TestNet。

接下来，我们创建一个新的空钱包，并将其设置为接收我们的比特币➋。如前所述，比特币钱包包含一个或多个比特币地址，每个比特币地址由公钥和私钥组成。这里➌，bitcoinJ 库为我们创建了一对新的密钥对。然后，我们打印出生成的公钥地址和私钥 ➍➎。最后，我们将新的密钥对添加到钱包中 ➏。

**警告**

*通常，在使用 bitcoinJ 时，你应该每次运行程序时重用相同的钱包，并在每次程序启动/停止时加载/保存它，否则程序可能会丢失资金。这对简单的 hello-money 程序不是问题。但是，在构建更复杂的 bitcoinJ 程序之前，请阅读“使用比特币钱包时的陷阱”第 239 页。*

比特币应用不仅需要一个钱包，还需要一个区块链。以下几行代码为我们初始化一个新的区块链：

```
File file = new File("my-blockchain");➊
SPVBlockStore store = new SPVBlockStore(params, file);➋
BlockChain chain = new BlockChain(params, wallet, store);➌

```

由于区块链消耗大量空间，我们将其写入一个名为*my-blockchain*的文件➊。接下来，我们创建一个区块存储，它是一个管理我们庞大区块链数据的对象➋。BitcoinJ 提供了几种不同类型的区块存储，每种类型在特性和性能上都有不同的权衡。在这个示例中，我们将使用一个`SPVBlockStore`对象，它通常是大多数应用的最佳选择。

那么，你需要了解哪些权衡呢？最大的性能挑战是，任何与比特币相关的应用都必须处理官方比特币区块链的大小，这个区块链的体积已经超过 10GB。大多数比特币应用*真的*需要这 10GB 的区块链数据吗？

为了回答这个问题，让我们考虑区块链存在的原因。从简化的角度来看，比特币区块链负责两项主要工作：

1\. 确定网络中每个人拥有多少比特币

2\. 确定通过网络广播的新交易是否有效

对于第一个任务，区块链允许我们检查所有历史区块并汇总关于每个比特币地址的综合数据，了解每个地址中包含了多少资金。对于第二个任务，它允许我们检查由网络创建的新交易区块，然后验证这些区块是否包含适当的哈希信息，以证明它们是根据最新难度要求正确挖掘的区块。

但考虑一下区块链的第一个任务：大多数应用程序需要辨别每个钱包中的资金数量吗？不，大多数应用程序只需要确定*一个*或少数几个钱包中的资金数量。因此，并不需要全部的 10GB 数据。先知般的中本聪在他原始的比特币白皮书中能够预见到，在这种情况下，可能会有一种名为 *简化支付验证（SPV）* 的优化方法。

**注意**

*我们在第九章中也简要介绍了 SPV，讨论了不同类型的比特币钱包时。*

下面是关于 SPV 工作原理的简要回顾：如果你提前知道自己对一个单一钱包感兴趣，你可以在从比特币网络拉取整个历史区块链时，直接统计该钱包中的金额。到那时，你只需要存储区块的头部信息，并且在大多数情况下可以完全忽略旧区块中的信息，这正是 `SPVBlockStore` 所做的事情。通过这样做，`SPVBlockStore`（截至 2014 年）的大小不到 1GB，只有官方区块链的十分之一，这就是为什么我们使用 `SPVBlockChain` 来存储我们的数据。

一旦我们创建了区块存储，我们就可以用它来创建一个 `BlockChain` 对象 ➌。请注意，当我们创建这个 `BlockChain` 对象时，我们必须传入我们创建的钱包。因为我们不需要下载全部的 10GB 数据，区块链对象需要提前知道哪些钱包（及其地址）对我们来说很重要，以便它可以选择正确的区块链数据进行下载。

**注意**

*尽管 SPVBlockStore 的大小远小于完整的区块链，但仍然可能需要很长时间才能从网络下载所有需要的数据——通常大约需要 20 分钟。然而，它会将这些数据写入文件，并且 SPVBlockStore 对象足够智能，可以检查提供的文件，看看自上次运行程序以来是否有数据已经下载。如果有，它只会下载自程序上次运行以来到达的新数据。*

#### *连接到比特币网络*

拥有一个钱包和存储区块链数据的位置后，我们现在可以连接到实际的比特币网络。比特币节点通过连接到几个半随机的对等节点来连接到比特币网络。以下是启动与多个对等节点连接的代码：

```
PeerGroup peerGroup = new PeerGroup(params, chain);➊
peerGroup.addPeerDiscovery(new DnsDiscovery(params));➋
peerGroup.addWallet(wallet);➌
peerGroup.start();➍
peerGroup.downloadBlockChain();➎

```

首先，我们创建一个`PeerGroup`对象 ➊ 来管理这些连接。接下来，我们选择一些随机的节点进行连接。我们通过向`PeerGroup`添加一个节点发现算法来实现这一点 ➋。`DnsDiscovery`类基本上使用一些经过验证的、可信的节点的 URL 作为起点，来发现愿意接受新连接的节点。然后，我们将钱包添加到`PeerGroup`对象中 ➌。

现在，我们终于准备好将应用程序注入到比特币网络中了！我们通过调用`PeerGroup.start` ➍来实现，这将找到并连接到一些节点，并通过网络套接字执行适当的握手操作。而且，就像任何比特币节点一样，我们要求对等节点向我们发送区块链数据，以便我们能够成为一个完全功能的节点 ➎。正如之前所提到的，这一步骤需要一些时间来运行，但仅在我们第一次运行程序时需要。

#### *监听新资金*

我们需要向`hello-money`程序添加的最后一个功能是一个钩子，用于检测资金何时到达：

```
wallet.addEventListener(new AbstractWalletEventListener()➊
    {
        public void onCoinsReceived(Wallet wallet, Transaction tx,➋
           BigInteger prevBalance, BigInteger newBalance)
        {
            System.out.println("Hello Money! Balance: "
                + newBalance + " satoshis");
        }
    });

```

bitcoinJ 钱包对象具有一个`addEventListener`成员函数，我们可以创建一个匿名的`EventListener`类，来拦截并监听可能发生在钱包上的不同事件 ➊。在我们的应用程序中，我们关心的是`onCoinsReceived`函数 ➋，它将在每次收到资金时被调用。让我们更详细地探讨一下这究竟意味着什么。

由于该程序直接运行在比特币网络中，它可以监听*比特币火流*，这是一个包含全球任何地方发生的每一笔比特币交易的实时数据流。每一笔交易都会被检查，以确定它是否涉及到向我们钱包中的任何比特币地址接收资金。在我们的应用程序中，钱包只包含一个地址。只要这笔交易到达（即使它还没有被纳入挖矿区块），我们的函数`onCoinsReceved`就会被调用。

**注意**

*在 hello-money 程序中，我们不需要担心捕获已接收资金的确认事件；我们只会监听新交易（未确认交易）的传输。然而，如果我们对确认感兴趣，我们可以通过 onTransactionConfidenceChanged 函数捕获它们。因为我们运行的是完整的比特币客户端，我们可以做任何事情，而在附录 A 中，我们*被迫*仅查看已确认的交易，这是由于 JSON-RPC 接口的限制。*

`onCoinsReceived` 函数有四个传入参数 ➋：钱包对象、交易对象、钱包中的之前余额和新的余额。bitcoinJ 库使用 Java 的 `BigInteger` 类来编码比特币余额，因为这种数值类型能够精确处理非常大的整数。如果你之前写过金融软件，你就知道为什么使用 `BigInteger` 类（或者你可能还记得电影 *Office Space* 中如何策划银行抢劫）。事实上，由于四舍五入错误，金融交易很容易搞砸，而使用大且精确的整数可以防止这个问题。因此，bitcoinJ 在进行所有比特币数学运算时，都会使用 satoshi——比特币的最小单位，价值为比特币的亿分之一。

**注意**

*因为我们在* *下载初始区块链后添加了事件监听器，所以 onCoinsReceived 函数只有在程序运行时出现新交易时才会被调用。如果我们在下载初始区块链之前声明它，bitcoinJ 的设计会导致 onCoinsReceived 也会在相关历史交易中被调用。*

最后，我们将程序置于一个无限循环中，这样程序会持续运行，直到我们等待钱款到账：

```
while(true){}

```

### 运行和测试 hello-money Java 程序

我们准备好运行并测试程序了！和之前一样，我们首先编译程序，然后运行：

```
> mvn install
> mvn exec:java

```

当程序连接到比特币网络并下载区块链时，一些信息应该会显示出来。第一次运行程序时，这可能需要一些时间：

```
Public address: 16YavT6SmJCuJpZgzRa6XG9WefPEu2M45
Private key: L3eoA1rXiD8kWFUzdxw744NWjoZNB5BGsxhzVas6y5KJgVteZ4uD
Downloading block chain of size 265184\. This may take a while.
Chain download 1% done with 262532 blocks to go, block date Feb 1, 2009 5:09:55 PM
Chain download 2% done with 259880 blocks to go, block date Feb 22, 2009 11:32:14 PM
Chain download 3% done with 257228 blocks to go, block date Mar 18, 2009 9:59:38 PM
Chain download 4% done with 254576 blocks to go, block date Apr 11, 2009 4:27:52 PM
Chain download 5% done with 251924 blocks to go, block date May 4, 2009 9:23:54 AM
...
Done downloading block chain

```

区块链下载完成后，你可以测试 `hello-money` 程序，并从你喜欢的钱包应用发送少量资金。只需将 0.0002 BTC 发送到公开地址，并记录下私钥（我们将在后续的示例程序中使用这些资金）。程序应该能够检测到资金到账并显示类似以下信息：

```
Hello Money! Balance: 20000 satoshis

```

钱包中的新余额应该以 satoshis 显示（除以 100,000,000 即可看到该数值确实为 0.0002 BTC）。

你已经成功编写了一个 bitcoinJ 程序，它创建了一个比特币钱包，并报告任何收到的资金。现在，让我们编写第二个程序，使用新存储的资金！

### 拜拜，钱

现在，让我们编写一个全新的程序，可以从任意比特币地址*发送*资金。要创建一个新的 `bye-bye-money` 程序，请在顶层程序目录中运行以下命令：

```
mvn archetype:generate -DgroupId=byebyemoney -DartifactId=bye-bye-money
   -DarchetypeArtifactId=maven-archetype-quickstart -DinteractiveMode=false

```

然后，按照我们在`hello-money`示例中所做的，准确地对 *pom.xml* 文件进行相同的添加，只是将 `<mainClass>hellomoney.App</mainClass>` 这一行改为 `<mainClass>byebyemoney.App</mainClass>`。（这些步骤类似于我们在 “为 hello-money 创建一个启动项目” 中的操作，见第 228 页）

就像以前一样，打开文件*src/main/java/byebyemoney/App.java*，并将其内容替换为以下程序：

```
package byebyemoney;

import com.google.bitcoin.core.*;
import com.google.bitcoin.store.*;
import com.google.bitcoin.discovery.DnsDiscovery;
import java.util.concurrent.ExecutionException;
import java.io.File;
import java.math.BigInteger;

public class App
{
    public static void main( String[] args )
        throws BlockStoreException, AddressFormatException,
                    InterruptedException, ExecutionException
    {
        NetworkParameters params = NetworkParameters.prodNet();
        Wallet wallet = new Wallet(params);
        DumpedPrivateKey key = new DumpedPrivateKey(params,
               "L1vJHdDqQ5kcY5q4QoY124zD21UVgFe6NL2835mp8UgG2FNU94Sy");
        wallet.addKey(key.getKey());
        BlockChain chain = new BlockChain(params, wallet,
               new MemoryBlockStore(params));
     PeerGroup peerGroup = new PeerGroup(params, chain);
     peerGroup.addPeerDiscovery(new DnsDiscovery(params));
     peerGroup.addWallet(wallet);
     peerGroup.start();
     peerGroup.downloadBlockChain();
     BigInteger balance = wallet.getBalance();
     System.out.println("Wallet balance: " + balance);
     Address destinationAddress = new Address(params,
               "1BTCorgHwCg6u2YSAWKgS17qUad6kHmtQW");
     BigInteger fee=BigInteger.valueOf(10000);
     Wallet.SendRequest req = Wallet.SendRequest.to(
              destinationAddress,balance.subtract(fee));
     req.fee = fee;
     Wallet.SendResult result = wallet.sendCoins(peerGroup, req);
     if(result != null)
         {
             result.broadcastComplete.get();
             System.out.println("The money was sent!");
         }
     else
         {
             System.out.println("Something went wrong sending the money.");
         }
   }
}

```

新程序中的许多行与我们之前的`hello-money`程序共享，但让我们仔细看一下新的部分。

#### *导入私钥*

要从我们的程序发送资金，我们需要导入前一个示例中的比特币地址的私钥。下面是执行此操作的代码：

```
DumpedPrivateKey key = new DumpedPrivateKey(params,➊
          "L1vJHdDqQ5kcY5q4QoY124zD21UVgFe6NL2835mp8UgG2FNU94Sy");
wallet.addKey(key.getKey());➋
BlockChain chain = new BlockChain(params, wallet,➌
          new MemoryBlockStore(params));

```

在前几行中，我们显式地将一个新的、已有的私钥添加到我们的钱包中 ➊➋。这是与在`hello-money`程序中接收资金的比特币地址相关的私钥。你需要将这一行中显示的私钥替换为你在运行之前示例时写下的私钥。此外，在这个新程序中，我们不使用`SPVBlockStore`功能；相反，我们使用 bitcoinJ 的`MemoryBlockStore` ➌功能作为变体。这个块存储不会创建文件，但通过使用它，我们的程序每次运行时都需要重新下载区块链。（这也保证了 bitcoinJ 会将正确的余额分配给钱包。我们将在《使用 BitcoinJ 钱包时的陷阱》一节中讨论原因，见第 239 页。）

#### *发送资金*

现在让我们看看实际上发送资金的代码：

```
BigInteger balance = wallet.getBalance();➊
System.out.println("Wallet balance: " + balance);➋
Address destinationAddress = new Address(params,➌
     "1BTCorgHwCg6u2YSAWKgS17qUad6kHmtQW");
BigInteger fee = BigInteger.valueOf(10000);➍
Wallet.SendRequest req = Wallet.SendRequest.to(➎
     destinationAddress,balance.subtract(fee));
req.fee = fee;➏
Wallet.SendResult result = wallet.sendCoins(peerGroup, req);➐

```

首先，我们获取钱包中包含的余额 ➊ 并显示它 ➋。接下来，我们声明资金应该发送到的目标地址 ➌。在此示例中，我们输入比特币基金会的主要捐赠地址；你可以自由替换为你自己钱包的公开地址。

现在，发送比特币时最好包含交易费用，我们声明费用为 10,000 聪 ➍。接下来，我们创建一个`SendRequest`对象 ➎，这是一个结构体，用于保存我们正在发送的基本信息，包括目标地址和要发送的金额（即余额减去费用）。然后，我们在这个对象上设置费用 ➏，并发送我们的资金 ➐！

#### *确保资金传输*

如果我们尝试发送比我们拥有的更多资金，如果费用不足，或者如果在错误的时刻互联网连接中断，资金可能永远不会被网络接受。因此，我们需要编写代码，等待并确保我们发送的资金传输到网络。以下是我们为此添加的代码：

```
result.broadcastComplete.get();➊
System.out.println("The money was sent!");➋

```

代码的第一行 ➊ 获取一个 Java *future*对象，这表明发送的交易已经正确地广播到网络。（Java 中的标准，future 用于获取关于单独执行线程的信息——在此情况下，是监控与比特币网络通信的线程。）如果这一行没有抛出异常，则我们会显示一条消息，表示资金已发送 ➋。

#### *运行 bye-bye-money*

我们可以像往常一样运行`bye-bye-money`（记得输入你自己的私钥）：

```
> mvn install
> mvn exec:java

```

**比特币 J 中的异常类型**

在这个示例中，我们省略了一个功能：错误处理。主函数简单地重新抛出了一些不同的异常，处理在发送资金时可能出现的错误。这些包括以下异常：

• `BlockStoreException`：当无法创建区块存储时，会抛出此异常（最常见的情况是，使用写入文件的区块存储类型时，文件损坏导致无法创建）。

• `AddressFormatException`：当地址格式不正确时，会抛出此异常。

• `InterruptedException`：当发生网络连接问题时，会抛出此异常。

• `ExecutionException`：当我们使用 future 对象并且在其他线程中发生异常时，会抛出此异常（例如，当我们检查交易广播是否完成时）。

在更复杂的比特币应用中，你应该单独捕获所有这些异常类型，并为应用用户添加更具描述性的错误信息。

由于该程序在内存中处理区块链，因此你需要等待几分钟或更长时间才能完成（即使你重新运行它）。如果程序成功，你将看到消息*资金已发送！*，资金应该到达目标钱包。你还可以访问区块链信息网站（如 *[`blockchain.info/`](http://blockchain.info/)*），输入源地址或目标地址，查看交易的详细信息是否已成为公共记录的一部分。

恭喜！你现在已经了解了编写比特币应用的基础知识！

### 使用 Wallets 在 BitcoinJ 中的注意事项

对于新手来说，钱包和 `BlockChain` 对象在 bitcoinJ 中的工作方式可能非常令人困惑。如果你没有完全理解 bitcoinJ 的行为，bitcoinJ 可能会报告不正确的钱包余额。

这发生的原因是 bitcoinJ 针对 SPV 区块链的概念进行了优化。我们之前讨论过 SPV 区块链的性能优势，但由于它们只包含有限的区块链数据，你需要遵循一些基本规则，以确保它们在 bitcoinJ 中正常工作：

1\. 如果你的应用钱包中已经有余额，bitcoinJ 需要在从网络下载区块链之前，了解余额的数量*。

2\. 区块链加载完成后，bitcoinJ 将执行必要的任务，以确保钱包在网络上出现新交易时的准确性。

3\. 如果你使用的是支持保存到磁盘文件的区块存储类型，那么你的应用也需要负责将钱包保存到文件（同样，它还需要负责加载区块存储和钱包数据）。

正如我们在构造 `BlockChain` 对象时所看到的，bitcoinJ 期望应用程序传递一个钱包对象。这使得当下载的区块中发现相关的*历史*交易时，可以更新钱包，并确保规则 #1 被执行：*确保在事后不向钱包中添加额外的密钥，并期望钱包在不重新下载区块链的情况下正常工作*。

类似地，当我们初始化`PeerGroup`对象时，我们调用了`addWallet()`将我们的钱包添加到对等组中。通过这样做，bitcoinJ 保持钱包余额与比特币网络中出现的任何*新*交易同步，同时遵循规则#2。

为了确保遵循规则#3，你可以使用`Wallet.loadFromFile()`和`Wallet.saveToFile()`函数。同时，还有一个`Wallet.autoSaveToFile()`函数，它可以帮助加载和保存区块存储和钱包数据。要了解如何正确使用这些函数，请查看 bitcoinJ 库中的示例程序。

如果你记住之前列出的三个基本原则，你将避免大多数掌握 bitcoinJ 过程中可能遇到的陷阱。

### 结论

我们希望你享受了这次关于 bitcoinJ 编程的旅程，我们期待看到你所开发的任何精彩新应用。毕竟，你构建的应用可能会彻底改变人们在互联网时代与金钱互动的方式！
