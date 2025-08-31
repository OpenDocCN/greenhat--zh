# 第三章：构建基础设施

![](img/chapterart.png)

本章中，我们将设置后端攻击基础设施以及必要的工具，以忠实地重现和自动化几乎所有手动设置中的痛点。我们将使用两个框架：针对 Linux 目标的 Metasploit 和针对 Windows 主机的 SILENTTRINITY。

## 传统方法

旧的攻击基础设施搭建方式是将每个框架安装在一台机器上，并在它们前面放置一个 Web 服务器，通过简单的模式匹配规则来接收和路由流量。如图 3-1 所示，指向 */secretPage* 的请求将被转发到 C2 后端，而其他页面则返回看似无害的内容。

![f03001.png](img/f03001.png)

图 3-1：C2 后端示意图

Nginx Web 服务器是代理 Web 流量的流行选择，并且可以相对快速地进行调优。首先，我们使用经典的软件包管理器（此处为 `apt`）进行安装：

```
root@Lab:~/# apt install -y nginx
root@Lab:~/# vi /etc/nginx/conf.d/reverse.conf
```

然后，我们创建一个配置文件，描述我们的路由策略，如列表 3-1 所示。

```
#/etc/nginx/conf.d/reverse.conf

server {
  # basic web server configuration
  listen 80;

  # normal requests are served from /var/www/html
  root /var/www/html;
  index index.html;
  server_name `www.mydomain.com`;

  # return 404 if no file or directory match
  location / {
     try_files $uri $uri/ =404;
  }

  # /msf URL gets redirected to our backend C2 framework
  location /msf {
     proxy_pass https://192.168.1.29:8443;
     proxy_ssl_verify off;
     proxy_set_header Host $host;
     proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
  }
  # repeat previous block for other C2 backends
}
```

列表 3-1：带有 HTTP 重定向器的标准 Nginx 配置文件

前几个指令定义了包含普通查询所需网页的根目录。接下来，我们指示 Nginx 将我们希望重定向的 URL（从 `/msf` 开始）直接转发到我们的 C2 后端，这可以通过 `proxy_pass` 指令清晰看出。

然后，我们可以通过 EFF 的 Certbot 快速设置 Secure Shell (SSL) 证书，并拥有一个完全功能的带有 HTTPS 重定向的 Web 服务器：

```
root@Lab:~/# add-apt-repository ppa:certbot/certbot
root@Lab:~/# apt update && apt install python-certbot-nginx
root@Lab:~/# certbot --nginx -d `mydomain.com` **-d** `www.mydomain.com`

Congratulations! Your certificate and chain have been saved at...
```

这种方法完全没问题，唯一的问题是，调优 Nginx 或 Apache 服务器可能会变得乏味和繁琐，特别是当这台机器将面对目标时，它的波动性会显著增加。服务器总是距离一个 IP 封禁就可能被重启或甚至终止。

配置 C2 后端也不轻松。没有托管提供商会给你一个预装所有依赖项的 Kali 发行版。这完全得靠你自己，最好确保 Metasploit 的 Ruby 版本准确无误；否则，它会不断报错，让你开始怀疑自己的理智。几乎任何依赖特定高级功能的应用程序都存在类似问题。

## 容器与虚拟化

解决方案是将所有应用程序及其所有依赖项打包好，并确保安装和调优到正确的版本。当你启动一台新机器时，不需要再安装任何东西。你只需下载整个包并作为一个整体运行。这基本上就是容器技术的核心，它席卷了整个行业，并改变了软件的管理和运行方式。由于稍后我们会使用一些容器，所以在准备我们的小环境时，不妨花些时间解析容器的内部结构。

容器世界中有许多参与者，他们在不同的抽象层次上工作或提供不同的隔离特性，包括 containerd、runC、LXC、rkt、OpenVZ 和 Kata Containers。我将使用旗舰产品 Docker，因为我们稍后会在本书中遇到它。

为了简化容器化的概念，大多数专家将其比作虚拟化：“容器是轻量级虚拟机，只是它们共享宿主机的内核”，这句话通常可以在图 3-2 中的熟悉图像下看到。

![f03002.png](img/f03002.png)

图 3-2：容器的过度简化图示

这句话对大多数只是想尽快部署应用的程序员来说可能足够了，但黑客需要更多，渴望更多细节。我们有责任了解足够的技术知识，以便能在必要时突破其规则。将虚拟化与容器化进行比较，就像将飞机与公交车进行比较。是的，我们都同意它们的目的都是运输人员，但它们的物流方式不同。甚至，涉及的物理原理也不同。

*虚拟化* 在现有操作系统之上创建一个完全功能的操作系统。它会按照自己的启动顺序进行启动，并加载文件系统、调度程序、内核结构等所有内容。来宾系统认为它在真实硬件上运行，但实际上，每一个系统调用背后，虚拟化服务（例如 VirtualBox）会将所有低级操作（如读取文件或触发中断）转换成宿主机的语言，反之亦然。这就是为什么你可以在 Windows 机器上运行 Linux 客户机的原因。

*容器化* 是一种不同的范式，其中系统资源被隔离并通过 Linux 内核的三个强大特性：命名空间、联合文件系统和 cgroups，巧妙地保护起来。

### 命名空间

*命名空间* 是可以分配给 Linux 资源（如进程、网络、用户、挂载的文件系统等）的标签。默认情况下，给定系统中的所有资源共享相同的默认命名空间，因此任何普通的 Linux 用户都可以列出所有进程、查看整个文件系统、列出所有用户等等。

然而，当我们启动一个容器时，容器环境创建的所有新资源——进程、网络接口、文件系统等等——都会被分配一个不同的标签。它们会被*容器化*在自己的命名空间中，忽略该命名空间外部资源的存在。

这一概念的完美示例是 Linux 如何组织其进程。在启动时，Linux 启动 systemd 进程，该进程被分配进程 ID（PID）号 1。随后，这个进程会启动后续的服务和守护进程，如 NetworkManager、crond 和 sshd，它们会依次被分配递增的 PID 号，如下所示：

```
root@Lab:~/# pstree -p
systemd(1)─┬─accounts-daemon(777)─┬─{gdbus}(841)
           │                      └─{gmain}(826)
           ├─acpid(800)
           ├─agetty(1121)
```

所有进程都链接到由 systemd 领导的同一树状结构中，所有进程都属于同一个命名空间。因此，它们可以相互查看和交互——前提是它们有权限这么做。

当 Docker（或更准确地说是 runC，负责启动容器的低级组件）启动一个新容器时，它首先在默认命名空间中执行自己（在 图 3-3 中是 PID 5），然后在新命名空间中启动子进程。第一个子进程在这个新命名空间中获得本地 PID 1，同时在默认命名空间中有一个不同的 PID（比如 6，如 图 3-3 所示）。

![f03003.png](img/f03003.png)

图 3-3：包含两个进程的新命名空间下的 Linux 进程树

新命名空间中的进程对外部环境发生的事情一无所知，但默认命名空间中的旧进程仍然可以完全看到整个进程树。这就是为什么在黑客攻击容器化环境时，主要挑战是打破这种命名空间隔离。如果我们能以某种方式在默认命名空间中运行一个进程，我们就能有效地监听主机上的所有容器。

容器内的每个资源继续与内核交互，而不经过任何中介。容器化的进程仅限于使用相同标签的资源。使用容器时，我们处于一个扁平但分隔的系统中，而虚拟化则像一组嵌套的俄罗斯套娃。

#### Metasploit 容器

让我们通过启动一个 Metasploit 容器来进行一个实际的例子。幸运的是，一个名为 phocean 的黑客已经创建了一个现成的镜像，我们可以在这个镜像上进行练习，地址在 [`github.com/phocean/dockerfile-msf/`](https://github.com/phocean/dockerfile-msf/)。当然，我们首先需要安装 Docker：

```
root@Lab:~/# curl -fsSL https://download.docker.com/linux/ubuntu/gpg   `| apt-key add -`

root@Lab:~/# add-apt-repository \
   **"deb [arch=amd64] https://download.docker.com/linux/ubuntu \**
   **$(lsb_release -cs) \**
   **stable"**

root@Lab:~/# apt update
root@Lab:~/# apt install -y docker-ce
```

然后我们下载 Docker 包或镜像，其中包含已经编译好并准备好的 Metasploit 文件、二进制文件和依赖项，可以通过 `docker pull` 命令来完成：

```
root@Lab:~/# docker pull phocean/msf
root@Lab:~/# docker run --rm -it phocean/msf
* Starting PostgreSQL 10 database server
[ OK ]
root@46459ecdc0c4:/opt/metasploit-framework#
```

`docker run` 命令将在新的命名空间中启动该容器的二进制文件。`--rm` 选项在容器终止时删除容器以清理资源。当测试多个镜像时，这个选项非常有用。`-it` 双重选项分配一个伪终端并链接到容器的 stdin 设备，以模拟交互式 Shell。

然后我们可以使用 `msfconsole` 命令启动 Metasploit：

```
root@46459ecdc0c4:/opt/metasploit-framework# ./msfconsole

       =[ metasploit v5.0.54-dev                          ]
+ -- --=[ 1931 exploits - 1078 auxiliary - 332 post       ]
+ -- --=[ 556 payloads - 45 encoders - 10 nops            ]
+ -- --=[ 7 evasion                                       ]

msf5 > **exit**
```

与从零开始安装 Metasploit 相比，你应该能够理解这两个命令节省了多少精力和时间。

当然，你可能会想，“在这个新的隔离环境中，我们如何从远程的 Nginx web 服务器访问监听器？”这是一个很好的问题。

启动容器时，Docker 会自动创建一对虚拟以太网（Linux 中为 `veth`）。可以将这些设备看作是物理电缆两端的两个连接器。一端被分配到新的命名空间，在该命名空间中，容器可以用来发送和接收网络数据包。这个 `veth` 通常在容器内被命名为熟悉的 `eth0`。另一端被分配到默认命名空间，并连接到一个网络交换机，该交换机负责与外部世界进行流量交换。Linux 将这个虚拟交换机称为 *网络桥接*。

在机器上快速运行 `ip addr` 命令，可以看到默认的 `docker0` 桥接器，分配了 172.17.0.0/16 的 IP 范围，准备分配给新的容器：

```
root@Lab:~/# ip addr
3: **docker0**: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 state group default
link/ether 03:12:27:8f:b9:42 brd ff:ff:ff:ff:ff:ff
inet 172.17.0.1/16 brd 172.17.255.255 scope global docker0
`--snip--`
```

每个容器都会从 `docker0` 桥接 IP 范围中获取一个专用的 `veth` 对，进而获得 IP 地址。

回到我们最初的问题，将流量从外部世界路由到容器，只需将流量转发到 Docker 网络桥接器，它会自动将流量送到正确的 `veth` 对。我们无需修改 iptables，只需调用 Docker 创建一个防火墙规则来实现这一点。在以下命令中，主机上的端口 8400 到 8500 将映射到容器中的端口 8400 到 8500：

```
root@Lab:~/# sudo docker run --rm \
**-it -p 8400-8500:8400-8500 \**
**-v ~/.msf4:/root/.msf4 \**
**-v /tmp/msf:/tmp/data \**
**phocean/msf**
```

现在，我们可以通过将数据包发送到主机的 IP 地址和相同端口范围，访问容器内监听任何端口（从 8400 到 8500）的处理程序。

在前一个命令中，我们还将主机上的目录 *~/.msf4* 和 */tmp/msf* 映射到容器中的目录 */root/.msf4* 和 */tmp/data*，这是一种在多次运行同一 Metasploit 容器时保留数据的实用技巧。

### 联合文件系统

这为我们引出了容器化的下一个概念——*联合文件系统* *(UFS)*，它通过将多个文件系统中的文件合并，呈现一个统一且一致的文件系统布局。让我们通过一个实际例子来探索它：我们将为 SILENTTRINITY 构建一个 Docker 镜像。

Docker 镜像在 *Dockerfile* 中定义。这是一个文本文件，其中包含构建镜像的指令，定义了要下载哪些文件，创建哪些环境变量，等等。这些命令非常直观，正如你在清单 3-2 中看到的那样。

```
# file: ~/SILENTTRINITY/Dockerfile
# The base Docker image containing binaries to run Python 3.7
FROM python:stretch-slim-3.7

# We install the git, make, and gcc tools
RUN apt-get update && apt-get install -y git make gcc

# We download SILENTTRINITY and change directories
RUN git clone https://github.com/byt3bl33d3r/SILENTTRINITY/ /root/st/
WORKDIR /root/st/

# We install the Python requirements
RUN python3 -m pip install -r requirements.txt

# We inform future Docker users that they need to bind port 5000
EXPOSE 5000

# ENTRYPOINT is the first command the container runs when it starts
ENTRYPOINT ["python3", "teamserver.py", "0.0.0.0", "stringpassword"]
```

清单 3-2：启动 SILENTTRINITY 团队服务器的 Dockerfile

我们首先构建一个 Python 3.7 的基础镜像，它是一个已经准备好并可用的文件和依赖集合，存放在官方 Docker 仓库 Docker Hub 中。接着，我们安装一些常用工具，如 `git`、`make` 和 `gcc`，这些工具稍后我们将用来下载代码库并运行团队服务器。`EXPOSE` 指令纯粹是用于文档目的。要实际暴露某个端口，我们仍然需要在执行 `docker run` 时使用 `-p` 参数。

接下来，我们使用一个指令拉取基础镜像，填充我们提到的工具和文件，并将生成的镜像命名为 `silent`：

```
 root@Lab:~/# docker build -t silent .
Step 1/7 : FROM python:3.7-slim-stretch
 ---> fad2b9f06d3b
Step 2/7 : RUN apt-get update && apt-get install -y git make gcc
 ---> Using cache
 ---> 94f5fc21a5c4
`--snip--`
Successfully built f5658cf8e13c
Successfully tagged silent:latest
```

每个指令都会生成一组新的文件，这些文件会被归在一起。这些文件夹通常存储在*/var/lib/docker/overlay2/*目录下，并以每个步骤生成的随机 ID 命名，类似于*fad2b9f06d3b*、*94f5fc21a5c4* 等。当镜像构建时，每个文件夹中的文件会被合并到一个新的单一目录下，称为*镜像层*。较高层次的目录会覆盖较低层次的目录。例如，在构建过程中，第 3 步中更改的文件会覆盖第 1 步中创建的同一文件。

当我们运行这个镜像时，Docker 会将镜像层以只读且 chroot 的文件系统形式挂载到容器内。为了允许用户在运行时修改文件，Docker 会在其上方进一步添加一个可写层，称为*容器层*或*upperdir*，如图 3-4 所示。

![f03004.png](img/f03004.png)

图 3-4：Docker 镜像的可写层。来源：[`dockr.ly/39ToIeq`](https://dockr.ly/39ToIeq)。

这就是赋予容器不可变性的原因。即使你在运行时覆盖了整个*/bin*目录，实际上你只会修改位于最上层的临时可写层，它掩盖了原始的*/bin*文件夹。当容器被删除时（记得`--rm`选项），可写层会被丢弃。构建镜像时准备的底层文件和文件夹将保持不变。

我们可以使用`-d`开关在后台启动新构建的镜像：

```
root@Lab:~/# docker run -d \
**-v /opt/st:/root/st/data \**
**-p5000:5000 \**
**silent**

3adf0cfdaf374f9c049d40a0eb3401629da05abc48c

# Connect to the team server running on the container
root@Lab:~st/# python3.7 st.py \wss://`username``:``strongPasswordCantGuess`**@192.168.1.29:5000**

[1] ST >>
```

完美。我们有了一个可用的 SILENTTRINITY Docker 镜像。为了能够从任何工作站下载它，我们需要将其推送到 Docker 仓库。为此，我们在 [`hub.docker.com`](https://hub.docker.com) 上创建一个帐户，并创建我们的第一个公共仓库，命名为*silent*。按照 Docker Hub 的约定，我们使用`docker tag`将 Docker 镜像重命名为`用户名`/`仓库名称`，然后将其推送到远程注册表，如下所示：

```
root@Lab:~/# docker login
Username: **sparcflow**
Password:

Login Succeeded

root@Lab:~/# docker tag silent sparcflow/silent
root@Lab:~/# docker push sparcflow/silent
```

现在，我们的 SILENTTRINITY Docker 镜像距离在我们未来启动的任何 Linux 机器上运行只差一个 `docker pull` 命令。

### Cgroups

容器的最后一个关键组件是*控制组（cgroups）*，它增加了一些命名空间无法解决的约束，例如 CPU 限制、内存、网络优先级以及容器可用的设备。正如它们的名字所示，cgroups 提供了一种通过对给定资源的相同限制来对进程进行分组和限制的方法；例如，属于/system.slice/accounts-daemon.service cgroup 的进程只能使用 30% 的 CPU 和 20% 的总带宽，并且无法访问外部硬盘。

这是命令`systemd-cgtop`的输出，它跟踪系统中 cgroup 的使用情况：

```
root@Lab:~/# systemd-cgtop
Control Group                            Tasks   %CPU   Memory  Input/s
/                                          188    1.1     1.9G        -
/docker                                      2      -     2.2M        -
/docker/08d210aa5c63a81a761130fa6ec76f9      1      -   660.0K        -
/docker/24ef188842154f0b892506bfff5d6fa      1      -   472.0K        -
```

当我们谈论 Docker 中的特权模式时，我们会回到 cgroups 的话题，所以现在先不展开讨论。

那么总结一下：无论我们选择哪个云服务提供商，以及他们托管的是什么 Linux 发行版，只要支持 Docker，我们就可以通过几条命令启动完全配置好的 C2 后端。接下来将运行我们的 Metasploit 容器：

```
root@Lab:~/# docker run -dit \
**-p 9990-9999:9990-9999 \**
**-v $HOME/.msf4:/root/.msf4 \**
**-v /tmp/msf:/tmp/data phocean/msf**
```

这将运行 SILENTTRINITY 容器：

```
root@Lab:~/# docker run -d \
**-v /opt/st:/root/st/data \**
**-p5000-5050:5000-5050 \**
**sparcflow/silent**
```

在这些示例中，我们使用的是 Metasploit 和 SILENTTRINITY 的原版版本，但我们也可以轻松添加自定义的 Boo-Lang 有效负载、Metasploit 资源文件等。最棒的是什么？我们可以根据需要复制我们的 C2 后端，轻松维护不同的版本，随意替换等等。挺酷的，对吧？

最后一步是将 Nginx 服务器“docker 化”，它会根据 URL 路径将请求路由到 Metasploit 或 SILENTTRINITY。

幸运的是，在这种情况下，大部分繁重的工作已经由@staticfloat 完成，他通过[`github.com/staticfloat/docker-nginx-certbot`](https://github.com/staticfloat/docker-nginx-certbot)使用 Let’s Encrypt 生成 SSL 证书，自动化了 Nginx 的设置。正如 Listing 3-3 所示，我们只需要对仓库中的 Dockerfile 做一些调整，以适应我们的需求，例如接受一个可变的域名和 C2 IP 来转发流量。

```
# file: ~/nginx/Dockerfile
# The base image with scripts to configure Nginx and Let's Encrypt
FROM staticfloat/nginx-certbot

# Copy a template Nginx configuration
COPY *.conf /etc/nginx/conf.d/

# Copy phony HTML web pages
COPY --chown=www-data:www-data html/* /var/www/html/

# Small script that replaces __DOMAIN__ with the ENV domain value, same for IP
COPY init.sh /scripts/

ENV DOMAIN=`"www.customdomain.com"`
ENV C2IP="192.168.1.29"
ENV CERTBOT_EMAIL="sparc.flow@protonmail.com"

CMD ["/bin/bash", "/scripts/init.sh"]
```

Listing 3-3：设置 Nginx 服务器并使用 Let’s Encrypt 证书的 Dockerfile

*init.sh* 脚本只是我们用来替换 Nginx 配置文件中字符串 `"__DOMAIN__"` 为环境变量`$DOMAIN`的几个`sed`命令，我们可以通过`-e`选项在运行时覆盖它。这意味着无论我们选择哪个域名，都可以轻松启动一个 Nginx 容器，它会自动注册正确的 TLS 证书。

Nginx 的配置文件几乎与 Listing 3-3 中的一样，所以我就不再重复讲解了。你可以查看构建这个镜像时涉及的所有文件，访问本书的 GitHub 仓库：[www.nostarch.com/how-hack-ghost](http://www.nostarch.com/how-hack-ghost/)。

启动一个完全功能的 Nginx 服务器，将流量重定向到我们的 C2 端点，现在只需一行命令：

```
root@Lab:~/# docker run -d \
```

`-p80:80 -p443:443 \`

```
**-e DOMAIN=**`"www.customdomain.com"` **\**
```

`-e C2IP="192.168.1.29" \` `-v /opt/letsencrypt:/etc/letsencrypt \` `sparcflow/nginx`

*www.<customdomain>.com*的 DNS 记录显然应该已经指向服务器的公共 IP，才能使这个操作成功。如果 Metasploit 和 SILENTTRINITY 容器可以在同一主机上运行，那么 Nginx 容器应该单独运行。可以把它看作是一个技术引信：只要遇到问题，它是第一个着火的。举个例子，如果我们的 IP 或域名被标记，我们只需重新启动一个新的主机并运行`docker run`命令。二十秒钟后，我们就有了一个新的域名和新的 IP，流量仍然会路由到相同的后端。

## IP 伪装

说到域名，我们来购买几个合法的域名，用来伪装我们的 IP。我通常喜欢购买两种类型的域名：一种用于工作站反向 shell，另一种用于机器。这个区分很重要。用户往往会访问看起来很正常的网站，因此可以购买一个看起来像是体育或烹饪博客的域名。像*experienceyourfood.com*这样的域名应该能起作用。  

然而，让一台服务器主动连接这个域名会显得很奇怪，因此购买的第二种域名应该像*linux-packets.org*这样的，伪装成一个合法的软件包分发点，托管一些 Linux 二进制文件和源代码文件。毕竟，服务器主动连接互联网下载软件包是被接受的模式。我无法计算有多少个威胁情报分析师因为网络中深处的服务器执行了`apt update`，从一个未知的主机下载了数百个软件包，而不得不丢弃那些误报。我们可以成为那个误报！

我不会再详细讲解域名注册，因为我们的目标不是通过网络钓鱼攻击进入公司，因此我们将避免讨论关于域名历史、分类、通过 DomainKeys Identified Mail（DKIM）进行的域名认证等问题。这些内容在我的书籍《如何像传奇一样黑客攻击》中有详细探讨。  

我们的基础设施现在几乎准备好了。我们还需要稍微调整我们的 C2 框架，准备好启动器，并启动监听器，但这些都将在后续的步骤中完成。  

## 自动化服务器设置

我们需要自动化的最后一个痛苦的步骤是设置实际的云服务器。无论每个提供商如何虚假宣称，仍然需要经历繁琐的菜单和标签设置，才能拥有一个正常工作的基础设施：防火墙规则、硬盘、机器配置、SSH 密钥、密码等等。  

这一步骤与云服务提供商本身紧密相关。像 AWS、微软 Azure、阿里巴巴和谷歌云平台这样的巨头通过大量强大的 API 完全支持自动化，而其他云服务提供商似乎对此漠不关心。幸运的是，这对我们来说可能不是一个大问题，因为我们通常管理的服务器只有三四台。我们可以轻松设置它们，或者从现有镜像克隆它们，只需三条`docker run`命令，就能拥有一个正常工作的 C2 基础设施。但如果我们能获取一张愿意与 AWS 共享的信用卡，我们也能自动化这最后一项繁琐的设置，从而涉及到任何现代技术环境中应该具备的基本要素之一：基础设施即代码。  

*基础设施即代码*的核心思想是拥有一个完整的声明性描述，涵盖任何时刻应运行的组件，从机器的名称到其上安装的最后一个包。然后，工具会解析此描述文件，并纠正任何观察到的不一致之处，例如更新防火墙规则、改变 IP 地址、附加更多磁盘，或者其他需要的操作。如果资源消失，它将被恢复以匹配所需的状态。听起来像是魔法，对吧？

多种工具可以帮助您实现这一自动化水平（无论是在基础设施层面还是操作系统层面），但我们将使用的工具是 HashiCorp 的 Terraform。

*Terraform*是开源的，支持多个云提供商（可以在[`registry.terraform.io`](https://registry.terraform.io)的文档中查看）。这使得它成为您选择接受 Zcash 的冷门云提供商时的最佳选择。本章的其余部分将专注于 AWS，因此您可以轻松地复制代码并学习如何使用 Terraform。

我想强调的是，开始时这一步是完全可选的。自动化设置两到三台服务器可能所花费的努力比它节省的时间还要多，因为我们已经拥有如此出色的容器设置，但自动化过程帮助我们探索当前的 DevOps 方法，以便在进入类似的环境时更好地理解我们需要关注什么。

Terraform 与所有 Golang 工具一样，是一个静态编译的二进制文件，因此我们不需要担心复杂的依赖关系。我们通过 SSH 连接到我们的跳板服务器，并立即下载该工具，如下所示：

```
root@Bouncer:~/# wget\
**https://releases.hashicorp.com/terraform/0.12.12/terraform_0.12.12_linux_amd64.zip**

root@Bouncer:~/# unzip terraform_0.12.12_linux_amd64.zip
root@Bouncer:~/# chmod +x terraform
```

Terraform 将使用我们提供的有效凭证与 AWS 云进行交互。前往 AWS IAM（身份与访问管理）——用户管理服务——创建一个程序化账户，并授予其对所有 EC2 操作的完全访问权限。*EC2*是 AWS 提供的管理机器、网络、负载均衡器等服务。若这是您第一次接触 AWS，您可以参考[`serverless-stack.com/chapters/`](https://serverless-stack.com/chapters/)上的逐步教程来创建 IAM 账户。

在 IAM 用户创建面板中，给予您新创建的用户程序化访问权限，如图 3-5 所示。

![f03005.png](img/f03005.png)

图 3-5：创建一个名为*terraform*的用户，并授予其访问 AWS API 的权限

通过附加 AmazonEC2FullAccess 策略（如图 3-6 所示），允许用户对 EC2 拥有完全控制权，以便管理机器。

![f03006.png](img/f03006.png)

图 3-6：将 AmazonEC2FullAccess 策略附加到*terraform*用户

将凭证下载为.*csv*文件。记下访问密钥 ID 和秘密访问密钥，如图 3-7 所示。接下来我们需要用到这些。

![f03007.png](img/f03007.png)

图 3-7：查询 AWS API 的 API 凭证

一旦获得了 AWS 访问密钥和秘密访问密钥，下载 AWS 命令行工具并保存您的凭证：

```
root@Bouncer:~/# apt install awscli

root@Bouncer:~/# aws configure
AWS Access Key ID [None]: **AKIA44ESW0EAASQDF5A0**
AWS Secret Access Key [None]: **DEqg5dDxDA4uSQ6xXdhvu7Tzi53**...
Default region name [None]: **eu-west-1**
```

然后我们设置一个文件夹来存放基础设施的配置：

```
root@Bouncer:~/# mkdir infra && cd infra
```

接下来，我们创建两个文件：*provider.tf*和*main.tf*。在前者中，我们初始化 AWS 连接器，加载凭证，并为我们打算创建的资源（例如`eu-west-1`（爱尔兰））分配默认区域，如下所示：

```
# provider.tf
provider "aws" {
  region  = "eu-west-1"
  version = "~> 2.28"
}
```

在*main.tf*中，我们将放置大部分架构的定义。Terraform 中的一个基本结构是*资源*——它描述了云服务提供商服务的离散单元，例如服务器、SSH 密钥、防火墙规则等。粒度的级别取决于云服务，并且可能迅速发展到令人难以理解的复杂程度，但这就是灵活性的代价。

要请求 Terraform 启动一个服务器，我们只需定义`aws_instance`资源，如下所示：

```
# main.tf
resource "aws_instance" "basic_ec2" {
  ami           = "ami-0039c41a10b230acb"
  instance_type = "t2.micro"
}
```

我们的`basic_ec2`资源是一个服务器，将启动由`ami-0039c41a10b230acb`标识的 Amazon 机器镜像（AMI），这恰好是一个 Ubuntu 18.04 镜像。你可以在[`cloud-images.ubuntu.com/locator/ec2/`](https://cloud-images.ubuntu.com/locator/ec2/)查看所有准备好的 Ubuntu 镜像。该服务器（或实例）类型为`t2.micro`，为其提供 1GB 内存和一个 vCPU。

我们保存*main.tf*并初始化 Terraform，以便它可以下载 AWS 提供程序：

```
root@Bounce:~/infra# terraform init
Initializing the backend...
Initializing provider plugins...
- Downloading plugin for provider "aws"

Terraform has been successfully initialized!
```

接下来，我们执行`terraform fmt`命令来格式化*main.tf*，然后执行`plan`指令来生成即将发生的基础设施变更列表，如下所示。你可以看到我们定义的属性已经安排好，服务器将会启动。相当酷。

```
root@Bounce:~/infra# terraform fmt && terraform plan
Terraform will perform the following actions:

  # aws_instance.basic_ec2 will be created
  + resource "aws_instance" "basic_ec2" {
      + ami                          = "ami-0039c41a10b230acb"
      + arn                          = (known after apply)
      + associate_public_ip_address  = (known after apply)
      + instance_type                = "t2.micro"
`--snip--`

Plan: 1 to add, 0 to change, 0 to destroy.
```

一旦我们验证了这些属性，就会调用`terraform apply`来在 AWS 上部署服务器。此操作还会在本地创建一个状态文件，描述我们刚刚创建的当前资源——一个服务器。

如果我们手动在 AWS 上终止服务器，并重新启动`terraform apply`，它将检测到本地状态文件和当前 EC2 实例状态之间的差异。它将通过重新创建服务器来解决这种差异。如果我们想启动另外九个相同配置的服务器，我们将`count`属性设置为`10`，然后再次运行`apply`。

尝试在 AWS（或任何云提供商）上手动启动和管理 10 个或 20 个服务器，你很快就会让头发变绿，脸涂白，开始在纽约市的街头跳舞。而我们其他使用 Terraform 的人，只需更新一个数字，如清单 3-4 所示，接着继续正常生活。

```
# main.tf launching 10 EC2 servers
resource "aws_instance" "basic_ec2" {
  ami           = "ami-0039c41a10b230acb"
  count         = 10
  instance_type = "t2.micro"
}
```

清单 3-4：使用 Terraform 创建 10 个 EC2 实例的最小代码

### 调整服务器

到目前为止，我们的服务器相当基础。让我们通过设置以下属性来对其进行优化：

+   一个 SSH 密钥，以便我们可以远程管理它，这对应于 Terraform 资源`aws_key_pair`。

+   一组防火墙规则——在 AWS 术语中称为*安全组*——用于控制哪些服务器可以相互通信以及如何通信。这是通过 Terraform 资源`aws_security_group`来定义的。安全组需要附加到*虚拟私有云（VPC）*，这是一种虚拟化的网络。我们只使用 AWS 创建的默认 VPC。

+   为每台服务器分配一个公共 IP。

列表 3-5 显示了设置了这些属性的*main.tf*。

```
# main.tf – compatible with Terraform 0.12 only

# We copy-paste our SSH public key
1 resource "aws_key_pair" "ssh_key" {
  key_name   = "mykey"
  public_key = "ssh-rsa AAAAB3NzaC1yc2EAAA..."
}

# Empty resource, since the default AWS VPC (network) already exists
resource "aws_default_vpc" "default" {
}

# Firewall rule to allow SSH from our bouncing server IP only
# All outgoing traffic is allowed
2 resource "aws_security_group" "SSHAdmin" {
  name        = "SSHAdmin"
  description = "SSH traffic"
  vpc_id      = aws_default_vpc.default.id
  ingress {
    from_port   = 0
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = ["123.123.123.123/32"]
  }
  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }
}

# We link the SSH key and security group to our basic_ec2 server

resource "aws_instance" "basic_ec2" {
  ami           = "ami-0039c41a10b230acb"
  instance_type = "t2.micro"

  vpc_security_group_ids     = aws_security_group.SSHAdmin.id
3 key_name                   = aws.ssh_key.id
  associate_public_ip_address= "true"
  root_block_device {
    volume_size = "25"
  }
}

# We print the server's public IP
output "public_ip " {
  value = aws_instance.basic_ec2.public_ip
}
```

列表 3-5：向*main.tf*添加一些属性

如前所述，`aws_key_pair`在 AWS 上注册一个 SSH 密钥，首次启动时会注入到服务器中。Terraform 上的每个资源稍后可以通过其 ID 变量进行引用，该变量在运行时填充——在本例中为`aws.ssh_key.id`。这些特殊变量的结构始终相同：`resourceType.resourceName.internalVariable`。

`aws_security_group`没有新颖之处，除了可能提到的默认 VPC（AWS 创建的默认虚拟网络段，类似于路由器接口）。防火墙规则只允许来自我们的跳板服务器的 SSH 流量。

我们再次运行`plan`命令，以确保所有属性和资源与预期结果匹配，如列表 3-6 所示。

```
root@Bounce:~/infra# terraform fmt && terraform plan
Terraform will perform the following actions:

  # aws_instance.basic_ec2 will be created
  + resource "aws_key_pair" "ssh_key2" {
      + id          = (known after apply)
      + key_name    = "mykey2"
      + public_key  = "ssh-rsa AAAAB3NzaC1yc2..."
    }

  + resource "aws_security_group" "SSHAdmin" {
      + arn                    = (known after apply)
      + description            = "SSH admin from bouncer"
      + id                     = (known after apply)
--`snip`--
   }

  + resource "aws_instance" "basic_ec2" {
      + ami                          = "ami-0039c41a10b230acb"
      + arn                          = (known after apply)
      + associate_public_ip_address  = true
      + id                           = (known after apply)
      + instance_type                = "t2.micro"
`--snip--`

Plan: 3 to add, 0 to change, 0 to destroy.
```

列表 3-6：检查属性是否已正确定义

Terraform 将创建三个资源。太好了。

最后一个细节，我们需要指示 AWS 在机器启动并运行时安装 Docker 并启动我们的容器 Nginx。AWS 利用`cloud-init`包，该包已安装在大多数 Linux 发行版中，用于在机器首次启动时执行脚本。这实际上就是 AWS 注入我们之前定义的公钥的方式。这个脚本被称为“用户数据”。

修改*main.tf*，添加 bash 命令以安装 Docker 并执行容器，如列表 3-7 所示。

```
resource "aws_instance" "basic_ec2" {
`--snip--`
1 user_data = <<EOF

#!/bin/bash
DOMAIN="www.linux-update-packets.org";
C2IP="172.31.31.13";

sleep 10
sudo add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
   $(lsb_release -cs) \
   stable"
apt update
apt install -y docker-ce
docker run -dti -p80:80 -p443:443 \
-e DOMAIN="www.customdomain.com" \
-e C2IP="$C2IP" \
-v /opt/letsencrypt:/etc/letsencrypt \
sparcflow/nginx

EOF
}
```

列表 3-7：从*main.tf*启动容器

EOF 块 1 包含一个多行字符串，便于注入由其他 Terraform 资源生成的环境变量的值。在这个例子中，我们硬编码了 C2 的 IP 和域名，但在实际情况下，这些将是负责启动后端 C2 服务器的其他 Terraform 资源的输出。

### 推向生产环境

我们现在准备通过简单的`terraform apply`将其推向生产环境，这将再次输出计划并请求手动确认，然后联系 AWS 创建所需的资源：

```
root@Bounce:~/infra# terraform fmt && terraform apply

aws_key_pair.ssh_key: Creation complete after 0s [id=mykey2]
aws_default_vpc.default: Modifications complete after 1s [id=vpc-b95e4bdf]
--`snip`--
aws_instance.basic_ec2: Creating...
aws_instance.basic_ec2: Creation complete after 32s [id=i-089f2eff84373da3d]

Apply complete! Resources: 3 added, 0 changed, 0 destroyed.
Outputs:

public_ip = 63.xx.xx.105
```

太棒了。我们可以使用默认的`ubuntu`用户名和私有 SSH 密钥 SSH 进入实例，确保一切正常运行：

```
root@Bounce:~/infra# ssh -i .ssh/id_rsa ubuntu@63.xx.xx.105

Welcome to Ubuntu 18.04.2 LTS (GNU/Linux 4.15.0-1044-aws x86_64)

ubuntu@ip-172-31-30-190:~$ **docker ps**
CONTAINER ID        IMAGE            COMMAND
5923186ffda5        sparcflow/ngi...   "/bin/bash /sc..."
```

完美。现在我们已经完全自动化了服务器的创建、设置和调优，我们可以释放内心的野性，复制这段代码，以生成任意数量的服务器，配备不同的防火墙规则、用户数据脚本以及其他任何设置。当然，更文明的方法是将我们刚写的代码封装在一个 Terraform 模块中，并根据需要传递不同的参数。具体细节，请查看书籍仓库中的*infra/ec2_module*，网址：[www.nostarch.com/how-hack-ghost](http://www.nostarch.com/how-hack-ghost/)。

我不会在这一已经很密集的章节中逐步讲解重构过程。重构主要是一些表面工作，比如在单独的文件中定义变量，创建多个安全组，将私有 IP 作为用户数据脚本中的变量传递，等等。我相信到现在为止，你已经掌握了足够的工作知识，可以从 GitHub 仓库中提取最终的重构版本，并随心所欲地进行实验。

本章的主要目标是向你展示如何在 60 秒内快速搭建一个功能齐全的攻击基础设施，因为这正是整个操作的核心：自动化的可重复性，这是任何点选操作所无法提供的。

我们只需几条命令即可部署我们的攻击服务器：

```
root@Bounce:~# git clone `your_repo`
root@Bounce:~# cd infra && terraform init
#update a few variables
root@Bounce:~# terraform apply
`--snip--`

Apply complete! Resources: 7 added, 0 changed, 0 destroyed.
Outputs:

nginx_ip_address = 63.xx.xx.105
c2_ip_address = 63.xx.xx.108
```

我们的基础设施终于准备好了！

## 资源

+   查看 Taylor Brown 的文章《将 Docker 带给 Windows 开发者，使用 Windows Server 容器》，链接：[`bit.ly/2FoW0nI`](http://bit.ly/2FoW0nI)。

+   在*[`bit.ly/2ZVRGpy`](http://bit.ly/2ZVRGpy)*上找到一篇关于容器运行时扩展的精彩文章。**

**Liz Rice 通过在她的讲座《在 Go 中从头构建一个容器》中实时编写代码，揭示了容器运行时的神秘面纱，这个讲座可以在 YouTube 上找到。* Scott Lowe 在[`blog.scottlowe.org/`](https://blog.scottlowe.org/)上提供了一个简短的网络命名空间实用介绍。* Jérôme Petazzoni 提供了更多关于命名空间、cgroups 和 UFS 的信息：可以在 YouTube 上观看。**
