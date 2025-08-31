# 第五章：容器镜像和运行时层

![image](img/common01.jpg)

要运行一个进程，我们需要存储空间。容器化软件的一个显著优势之一是能够将应用程序与其依赖项捆绑在一起进行交付。因此，我们需要存储程序的可执行文件以及其使用的任何共享库。我们还需要存储配置文件、日志和程序管理的任何数据。所有这些存储都必须隔离，以防容器干扰主机系统或其他容器。总的来说，这对存储需求很大，这意味着容器引擎必须提供一些独特的功能，以有效利用磁盘空间和带宽。在本章中，我们将探讨分层文件系统如何使容器镜像下载高效，并使容器启动高效。

### 文件系统隔离

在 第二章 中，我们看到如何使用 *chroot* 环境创建文件系统的一个独立隔离部分，该部分只包含我们运行进程所需的二进制文件和库。即使是运行简单的 `ls` 命令，我们也需要这个二进制文件和几个库。一个更完整功能的容器，比如运行 NGINX web 服务器的容器，需要更多——一个完整的 Linux 发行版的文件集。

在 chroot 示例中，当我们准备好使用它时，我们从主机系统构建了隔离的文件系统。对于容器而言，这种方法是不实际的。相反，隔离的文件系统被打包在一个 *容器镜像* 中，这是一个包含所有文件和元数据（如环境变量和默认可执行文件）的即用包。

#### 容器镜像内容

让我们快速查看一个 NGINX 容器镜像的内部。在本章中，我们将使用 Docker 运行命令，因为它仍然是构建容器镜像的最常用工具。

**注意**

*本书示例的示例存储库位于* [`github.com/book-of-kubernetes/examples`](https://github.com/book-of-kubernetes/examples)。*请参阅 第 xx 页 上的“运行示例”获取设置详细信息。*

在本章的示例中，从 *host01* 运行以下命令下载镜像：

```
root@host01:~# docker pull nginx
Using default tag: latest
latest: Pulling from library/nginx
...
Status: Downloaded newer image for nginx:latest
docker.io/library/nginx:latest
```

`docker pull` 命令会从 *镜像仓库* 下载一个镜像。镜像仓库是一个实现下载和发布容器镜像 API 的 Web 服务器。我们可以通过 `docker images` 命令列出已下载的镜像：

```
root@host01:~# docker images
REPOSITORY   TAG       IMAGE ID       CREATED       SIZE
nginx        latest    f0b8a9a54136   7 days ago    133MB
```

这个镜像大小为 133MB，并具有唯一标识符 `f0b8a9a54136`。（你的标识符将不同，因为每天都会构建新的 NGINX 容器镜像。）该镜像不仅包含 NGINX 可执行文件和所需的库，还包括基于 Debian 的 Linux 发行版。我们在 第一章 中简要看到了这一点，当时我们在 Ubuntu 主机和内核上演示了 Rocky Linux 容器，但让我们稍微详细地看一下。首先，运行一个 NGINX 容器：

```
root@host01:~# docker run --name nginx -d nginx
516d13e912a55cfc6f73f0dd473661d6b7d3b868d5a07a2bc7253971015b6799
```

`--name` 标志为容器指定了一个友好的名称，未来我们可以在命令中使用这个名称，而 `-d` 标志则将容器发送到后台运行。

现在，让我们探索一下运行中容器的文件系统：

```
root@host01:~# docker exec -ti nginx /bin/bash
root@516d13e912a5:/#
```

在这里，我们可以看到 NGINX 工作所需的各种库：

```
root@516d13e912a5:/# ldd $(which nginx)
        linux-vdso.so.1 (0x00007ffe2a1fa000)
...
        libc.so.6 => /lib/x86_64-linux-gnu/libc.so.6 (0x00007fe0d6531000)
        /lib64/ld-linux-x86-64.so.2 (0x00007fe0d6ed4000)
```

所有这些库都是我们下载的容器镜像的一部分，因此我们的 NGINX 容器不需要（也无法访问）主机系统中的任何文件。

我们不仅拥有了大量必要的库，而且在 */etc* 目录下还有典型的配置文件，这是我们期望在 Debian 系统中找到的：

```
root@516d13e912a5:/# ls -1 /etc
...
debian_version
deluser.conf
dpkg
...
systemd/
...
```

这个列表显示了文件系统中甚至包含了一些对容器来说并不真正需要的目录，比如 */etc/systemd* 目录。（记住，容器只是以隔离方式运行的一组相关进程，所以容器几乎从不运行像 systemd 这样的系统服务管理器。）这个完整的文件系统被包含进来有几个原因。首先，许多进程被设计成预期通常的文件集合会存在。其次，从一个典型的 Linux 发行版开始构建容器镜像要更容易一些。

我们容器的独立文件系统也是可写的。既然我们已经打开了这个 shell，让我们向容器中的一个文件发送一些随机数据，以便稍后从主机上检查这个存储。然后我们可以退出 shell：

```
root@516d13e912a5:/# dd if=/dev/urandom of=/tmp/data bs=1M count=10
...
10485760 bytes (10 MB, 10 MiB) copied, 0.0913977 s, 115 MB/s
root@516d13e912a5:/# exit
```

`dd` 命令将一个 10MB 的文件写入了 */tmp* 目录。尽管我们退出了 shell，容器仍然在运行，因此我们可以使用 `docker inspect` 查看该容器使用的磁盘空间：

```
root@host01:~# docker inspect -s nginx | jq '.[0].SizeRw'
10487109
```

`-s` 标志告诉 `docker inspect` 输出容器的大小。由于 `docker inspect` 生成了庞大的 JSON 输出，我们使用 JSON 查询工具 `jq` 来选择我们需要的字段。

报告的大小大约是 10MB，表明容器只消耗了写入的文件所需的读写存储空间，加上 NGINX 写入的任何文件。我们将在本章的后续部分更详细地探讨这一点。

#### 镜像版本与层

快速下载一个预先打包好的文件系统以运行一个进程，仅仅是容器镜像的一个优点。另一个优点是能够标记镜像的不同版本，以便于快速升级。让我们通过拉取并运行两个不同版本的 Redis，这个流行的内存键值数据库，来探索一下这一点：

```
   root@host01:~# docker pull redis:6.0.13-alpine
   6.0.13-alpine: Pulling from library/redis
➊ 540db60ca938: Pull complete 
   29712d301e8c: Pull complete 
   8173c12df40f: Pull complete 
   ...
   docker.io/library/redis:6.0.13-alpine
   root@host01:~# docker pull redis:6.2.3-alpine
   6.2.3-alpine: Pulling from library/redis
➋ 540db60ca938: Already exists 
   29712d301e8c: Already exists 
   8173c12df40f: Already exists 
   ...
   docker.io/library/redis:6.2.3-alpine
```

冒号后的数据是*镜像标签*，作为版本标识符。之前，当我们省略这个标签时，Docker 默认为*latest*，它是一个像其他标签一样的标签，但根据约定，用于指代最新发布的镜像。通过指定版本，我们可以确保即使发布了更新的 Redis 版本，我们仍然会继续运行相同的版本，直到我们准备好升级为止。标签可以包含任何字符，通常在连字符后添加额外的信息。在这个例子中，标签末尾的 `-alpine` 表示这个镜像基于 Alpine Linux，这是一个轻量级的 Linux 发行版，因为其小巧的体积，它在制作容器镜像时非常流行。

另一个值得注意的有趣事项是，当我们下载 Redis 的第二个版本时，其中一些内容 ➋ 被标记为 `已存在`。查看第一个 Redis 下载，我们可以看到相同的唯一标识符也出现在那里 ➊。这是因为一个容器镜像是由多个层组成的，这些标识符唯一地描述了一个层。如果我们已经下载的层被另一个镜像使用，我们就不需要再次下载它，从而节省了下载时间。此外，每个层只需要在磁盘上存储一次，从而节省了磁盘空间。

我们现在已经下载了两个不同版本的 Redis：

```
root@host01:~# docker images | grep redis
redis        6.0.13-alpine   a556c77d3dce   2 weeks ago   31.3MB
redis        6.2.3-alpine    efb4fa30f1cf   2 weeks ago   32.3MB
```

虽然 Docker 报告每个镜像的大小大约为 30MB，但这是所有层的总大小，并未考虑共享层所带来的存储节省。实际存储在磁盘上的空间更少，正如我们通过检查 Docker 使用磁盘空间的情况所看到的：

```
root@host01:~# docker system df -v
Images space usage:

REPOSITORY TAG           ... SIZE      SHARED SIZE   UNIQUE SIZE ...
redis      6.0.13-alpine ... 31.33MB   6.905MB       24.42MB     ...
redis      6.2.3-alpine  ... 32.31MB   6.905MB       25.4MB      ...
```

这两个 Redis 镜像共享了将近 7MB 的基础层。

这两个版本的 Redis 可以分别运行：

```
root@host01:~# docker run -d --name redis1 redis:6.0.13-alpine
66dbf56ec0e8db24ca78afc07c68b7d0699d68b4749e0c03310857cfce926366
root@host01:~# docker run -d --name redis2 redis:6.2.3-alpine
9dd3f86a1284171e5ca60f7f8a6a13dc517237826a92b3cb256f5ac64a5f5c31
```

现在两个镜像都在运行，我们可以确认我们的容器中有我们想要的确切版本的 Redis，与最新发布的版本无关，也不受主机服务器上可用版本的影响：

```
root@host01:~# docker logs redis1 | grep version
1:C 21 May 2021 14:18:24.952 # Redis version=6.0.13, ...
root@host01:~# docker logs redis2 | grep version
1:C 21 May 2021 14:18:36.387 # Redis version=6.2.3, ...
```

这对于构建可靠系统来说是一个很大的优势。我们可以使用一个版本的软件彻底测试我们的应用程序，并确保在我们选择升级之前，该版本会继续使用。我们还可以轻松地将软件与新版本进行测试，而不需要升级主机系统。

### 构建容器镜像

在前面的例子中，我们看到通过共享层来减少容器镜像的下载和磁盘需求。这种层共享可以与任何容器镜像一起使用，而不仅仅是同一软件的两个不同版本。

容器镜像中的层来自于它的构建方式。容器镜像的构建从一个*基础镜像*开始。例如，我们的两个 Redis 版本都是从相同的 Alpine Linux 基础镜像开始的，这也是为什么这些层在该镜像中得以共享的原因。从基础镜像开始，构建过程中的每个步骤都会产生一个新的层。这个新层仅包含来自该构建步骤的文件系统变化。

基础镜像也必须来自某个地方，最终必须有一个初始层，这通常是从某个 Linux 发行版创建的最小 Linux 文件系统，转移到一个空的容器镜像中，然后扩展成为初始层。

#### 使用 Dockerfile

构建容器镜像有许多不同的方法，但最流行的方法是创建一个名为 *Dockerfile* 或 *Containerfile* 的文件，指定镜像的命令和配置。以下是一个简单的 *Dockerfile*，它将 web 内容添加到 NGINX 镜像中：

*Dockerfile*

```
---
FROM nginx

# Add index.html
RUN echo "<html><body><h1>Hello World!</h1></body></html>" \
    >/usr/share/nginx/html/index.html
```

每一行 *Dockerfile* 中都以一个命令开始，后面跟着参数。空行和 `#` 后的内容会被忽略，行末的反斜杠表示该命令会延续到下一行。命令有很多种，以下是最常见的几种：

FROM 指定此构建的基础镜像。

RUN 在容器内运行命令。

COPY 将文件复制到容器中。

ENV 指定一个环境变量。

ENTRYPOINT 配置容器的初始进程。

CMD 设置初始进程的默认参数。

Docker 提供了 `docker build` 命令来从 *Dockerfile* 构建镜像。`docker build` 命令通过逐一运行 *Dockerfile* 中的每个命令来创建一个新镜像。列表 5-1 说明了如何运行 `docker build`。

```
   root@host01:~# cd /opt/hello
   root@host01:/opt/hello# docker build -t hello .
➊ Sending build context to Docker daemon  2.048kB
   Step 1/2 : FROM nginx
 ➋ ---> f0b8a9a54136
   Step 2/2 : RUN echo "<html><body><h1>Hello World!</h1></body></html>" ...
 ➌ ---> Running in 77ba9163d0a5
   Removing intermediate container 77ba9163d0a5
    ---> e9ca31d590f9
   Successfully built e9ca31d590f9
➍ Successfully tagged hello:latest
```

*列表 5-1: Docker 构建*

`-t` 开关告诉 `docker build` 将构建过程中的镜像存储为名称为 `hello` 的镜像。

审查构建过程中的各个步骤将有助于澄清容器镜像是如何创建的。首先，Docker 将*构建上下文*发送到 Docker 守护进程 ➊。构建上下文是一个目录及其所有文件和子目录。在这种情况下，当我们在 `docker build` 命令末尾添加 `.` 时，我们指定了当前目录作为构建上下文。实际的容器镜像构建是在守护进程内进行的，因此只有构建上下文中的文件才能用于 `COPY` 命令。

其次，Docker 确定了我们的基础镜像，在本例中是 `nginx`。它显示的唯一标识符 ➋ 与我们运行 `docker images` 时之前显示的 NGINX 镜像相同。第三，Docker 执行了我们在 `RUN` 步骤中指定的命令。该命令实际上是在基于我们的 NGINX 基础镜像 ➌ 创建的容器内运行的，这意味着只有容器镜像中安装的命令才能运行。如果我们需要其他命令可用，可能需要创建一个 `RUN` 步骤来安装它们，才能使用。

所有构建步骤完成后，Docker 使用 `-t` 标志用我们提供的名称为新容器镜像打标签。如前所述，我们没有指定版本，因此默认使用 `latest`。现在我们可以在可用镜像列表中看到该镜像：

```
root@host01:/opt/hello# docker images | grep hello
hello        latest          e9ca31d590f9   9 minutes ago   133MB
```

这个镜像的唯一标识符与 Listing 5-1 结尾处的输出匹配。这个镜像显示为 133MB，因为它包含了所有来自 NGINX 镜像的层，外加我们添加的新小 HTML 文件。和之前一样，共享层只存储一次，因此构建这个镜像所需的额外存储非常小。

**注意**

*当你自己尝试这个例子时，显示的“hello”镜像的唯一标识符会有所不同，即使 Dockerfile 中的 HTML 文件内容相同。每一层的标识符不仅基于该层的文件内容，还基于其上层的标识符。因此，如果两个镜像的标识符相同，我们可以确定它们的内容完全相同，即使它们是分开构建的。*

我们可以像运行其他镜像一样运行基于这个新镜像的容器：

```
root@host01:/opt/hello# docker run -d -p 8080:80 hello
83a23cf2921bb37474bfcefb0da45f9953940febfefd01ebadf35405d88c4396
root@host01:/opt/hello# curl http://localhost:8080/
<html><body><h1>Hello World!</h1></body></html>
```

如第一章所述，`-p`标志将主机端口转发到容器，使我们即使容器运行在一个独立的网络命名空间中，仍然可以从主机访问 NGINX 服务器。然后我们可以使用`curl`来查看我们的容器是否包含我们提供的内容。

#### 镜像标记与发布

镜像已经可以在本地运行，但我们还没有准备好将其发布到注册表。要发布到注册表，我们需要给镜像一个名称，该名称包含注册表位置的完整主机和路径，以确保我们在引用镜像时能获取到我们期望的内容。

为了演示，让我们从不同的注册表拉取多个 BusyBox 镜像。我们将从*quay.io*开始，*quay.io*是一个替代的容器镜像注册表：

```
root@host01:/opt/hello# docker pull quay.io/quay/busybox
...
quay.io/quay/busybox:latest
```

这个镜像名称指定了主机`quay.io`以及该主机内镜像的位置`quay/busybox`。和之前一样，因为我们没有指定版本，`latest`被用作默认版本。我们能够拉取名为`latest`的版本，因为有人明确将`latest`版本的镜像发布到了这个注册表。

我们使用这个命令获取的 BusyBox 镜像与直接拉取`busybox`时获得的镜像不同：

```
root@host01:/opt/hello# docker pull busybox
...
docker.io/library/busybox:latest
root@host01:/opt/hello# docker images | grep busybox
busybox                latest          d3cd072556c2   3 days ago       1.24MB
quay.io/quay/busybox   latest          e3121c769e39   8 months ago     1.22MB
```

当我们使用简单名称`busybox`时，Docker 默认从`docker.io/library`拉取镜像。这个注册表被称为*Docker Hub*，你可以在*[`hub.docker.com`](https://hub.docker.com)*浏览它。

类似地，当我们使用简单名称`hello`构建镜像时，Docker 会将其视为属于`docker.io/library`。这个路径是官方 Docker 镜像的路径，当然，我们没有权限将镜像发布到这里。

本章的自动化设置包括运行一个本地容器注册表，这意味着如果我们正确命名镜像，我们可以将其发布到该本地注册表：

```
root@host01:/opt/hello# docker tag hello registry.local/hello
root@host01:/opt/hello# docker images | grep hello
hello                  latest          e9ca31d590f9   52 minutes ago   133MB
registry.local/hello   latest          e9ca31d590f9   52 minutes ago   133MB
```

现在，同一个镜像在两个不同的名称下存在，利用镜像按层存储的方式提供了额外的优势。为镜像添加额外的名称是很便宜的。当然，我们本来也可以在最初运行`docker build`时使用完整的名称，但在构建和本地使用镜像时，使用较短的名称更为方便。

现在我们已经正确命名了镜像，我们可以使用`docker push`将其发布：

```
root@host01:/opt/hello# docker push registry.local/hello
Using default tag: latest
The push refers to repository [registry.local/hello]
...
```

我们的本地注册表一开始是空的，因此此命令会上传所有的层，但如果我们推送任何包含相同层的未来镜像，它们不会被再次上传。同样，如果我们从注册表中删除一个镜像标签，层数据并不会被删除。

发布镜像的能力不限于我们自己构建的镜像。我们可以标记并推送刚刚从 Docker Hub 下载的 BusyBox 镜像：

```
root@host01:/opt/hello# docker tag busybox registry.local/busybox
root@host01:/opt/hello# docker push registry.local/busybox
Using default tag: latest
The push refers to repository [registry.local/busybox]
...
root@host01:/opt/hello# cd
```

重新标记一个镜像，以便我们可以将其上传到私有注册表，这是一个常见的做法，它有助于应用程序更快启动并避免依赖于互联网注册表。

最后一个命令（`cd`）将我们带回到我们的主目录，因为我们在*/opt/hello*中已经完成操作。

### 镜像和容器存储

如前所述，使用单独的层来构建容器镜像有多个优势，包括减少下载大小、减少磁盘空间，并且可以在不使用额外空间的情况下为镜像重新标记新名称。运行中的容器所需的额外磁盘空间仅限于我们在容器运行时写入的文件。最后，所有的示例都展示了新容器启动的速度有多快。所有这些特性加在一起，说明为什么层必须共享，不仅仅是镜像，还有新的容器。为了更好地利用这种分层方法来构建高效的镜像，了解这种分层文件系统是如何工作的非常重要。

#### 覆盖文件系统

当我们运行容器时，我们看到的是一个看似单一的文件系统，所有层都被合并在一起，并且可以对任何文件进行更改。如果我们从同一个镜像运行多个容器，我们会在每个容器中看到一个独立的文件系统，这样一个容器中的更改不会影响到另一个容器。那么，为什么在每次启动容器时不需要复制整个文件系统呢？答案就是*覆盖文件系统*。

一个 Overlay 文件系统有三个主要部分。*lower 目录* 是“基础”层所在的位置。（可能有多个 lower 目录。）*upper* 目录包含“覆盖”层，*mount* 目录是将统一文件系统提供给用户的地方。挂载目录中的目录列表反映了所有层的文件，按优先顺序排列。对挂载目录所做的任何更改，实际上是通过从 lower 目录将更改的文件复制到 upper 目录并更新它来写入上层目录——这一过程称为 *写时复制*。删除操作也会作为元数据写入 upper 目录，因此 lower 目录保持不变。这意味着多个用户可以共享 lower 目录而不会发生冲突，因为它仅供读取，从不写入。

Overlay 文件系统不仅对容器镜像和容器有用，它还对嵌入式系统有用，例如网络路由器，对于这种设备，固件中写入只读文件系统，使得设备每次重新启动时都能安全地回到已知状态。它对于虚拟机也有用，可以使多个虚拟机从同一镜像启动。

Overlay 文件系统是由 Linux 内核模块提供的，能提供非常高的性能。我们可以轻松创建一个 Overlay 文件系统。第一步是创建必要的目录：

```
root@host01:~# mkdir /tmp/{lower,upper,work,mount}
```

`mkdir` 命令在 */tmp* 目录中创建了四个独立的目录。我们已经讨论过 *lower* 目录、*upper* 目录和 *mount* 目录。*work* 目录是一个额外的空目录，Overlay 文件系统使用它作为临时空间，确保挂载目录中的更改是原子性的——也就是说，确保它们一次性出现。

让我们向 lower 目录和 upper 目录中添加一些内容：

```
root@host01:~# echo "hello1" > /tmp/lower/hello1
root@host01:~# echo "hello2" > /tmp/upper/hello2
```

接下来，我们只需要挂载 Overlay 文件系统：

```
root@host01:~# mount -t overlay \
  -o rw,lowerdir=/tmp/lower,upperdir=/tmp/upper,workdir=/tmp/work \
  overlay /tmp/mount
```

*/tmp/mount* 目录现在包含了上层和下层目录的合并内容：

```
root@host01:~# ls -l /tmp/mount
total 8
-rw-r--r-- 1 root root 7 May 24 23:05 hello1
-rw-r--r-- 1 root root 7 May 24 23:05 hello2
root@host01:/opt/hello# cat /tmp/mount/hello1
hello1
root@host01:/opt/hello# cat /tmp/mount/hello2
hello2
```

我们所做的任何更改都会显示在挂载位置，但实际上是在上层目录中进行的：

```
root@host01:~# echo "hello3" > /tmp/mount/hello3
root@host01:~# ls -l /tmp/mount
total 8
-rw-r--r-- 1 root root 7 May 24 23:05 hello1
-rw-r--r-- 1 root root 7 May 24 23:10 hello2
-rw-r--r-- 1 root root 7 May 24 23:09 hello3
root@host01:~# ls -l /tmp/lower
total 4
-rw-r--r-- 1 root root 7 May 24 23:05 hello1
root@host01:~# ls -l /tmp/upper
total 8
-rw-r--r-- 1 root root 7 May 24 23:10 hello2
-rw-r--r-- 1 root root 7 May 24 23:09 hello3
```

此外，即使删除文件，也不会影响 lower 目录：

```
   root@host01:~# rm /tmp/mount/hello1
   root@host01:~# ls -l /tmp/mount
   total 8
   -rw-r--r-- 1 root root 7 May 24 23:10 hello2
   -rw-r--r-- 1 root root 7 May 24 23:09 hello3
   root@host01:~# ls -l /tmp/lower
   total 4
   -rw-r--r-- 1 root root 7 May 24 23:05 hello1
   root@host01:~# ls -l /tmp/upper
   total 8
➊ c--------- 1 root root 0, 0 May 24 23:11 hello1
   -rw-r--r-- 1 root root    7 May 24 23:10 hello2
   -rw-r--r-- 1 root root    7 May 24 23:09 hello3
```

`hello1` 在上层目录 ➊ 的列表旁边的 `c` 表明这是一个 *字符特殊文件*。它的作用是表示该文件在上层目录中已被删除。因此，尽管它仍然存在于 lower 目录中，但在挂载的文件系统中并未显示出来。

多亏了这种方法，我们可以使用独立的 Overlay 重新使用 lower 目录，类似于我们可以从同一镜像运行多个独立容器的方式：

```
root@host01:~# mkdir /tmp/{upper2,work2,mount2}
root@host01:~# mount -t overlay \
  -o rw,lowerdir=/tmp/lower,upperdir=/tmp/upper2,workdir=/tmp/work2 \
  overlay /tmp/mount2
root@host01:~# ls -l /tmp/mount2
total 4
-rw-r--r-- 1 root root 7 May 24 23:05 hello1
```

不仅来自 lower 目录的“已删除”文件会出现，来自第一个上层目录的任何内容也不会出现，因为它不是这个新 Overlay 的一部分。

#### 理解容器层

有了关于 Overlay 文件系统的信息，我们可以探索正在运行的 NGINX 容器的文件系统：

```
root@host01:~# ROOT=$(docker inspect nginx \
  | jq -r '.[0].GraphDriver.Data.MergedDir')
root@host01:~# echo $ROOT
/var/lib/docker/overlay2/433751e2378f9b11.../merged
```

如前所述，我们使用 `jq` 只选择我们想要的字段；在这种情况下，它是容器文件系统的 *merged* 目录路径。这个合并目录是 overlay 文件系统的挂载点：

```
root@host01:~# mount | grep $ROOT | tr [:,] '\n'
overlay on /var/lib/docker/overlay2/433751e2378f9b11.../merged ...
lowerdir=/var/lib/docker/overlay2/l/ERVEI5TCULK4PCNO2HSWB4MFDB
/var/lib/docker/overlay2/l/RQDO2PYQ3OKMKDY3DAYPAJTZHF
/var/lib/docker/overlay2/l/LFSBVPYPODQJXDL5WQTI7ISYNC
/var/lib/docker/overlay2/l/TLZUYV2BFQNPFGU3AZFUHOH27V
/var/lib/docker/overlay2/l/4M66FKSHDBNUWE7UAF2REQHSB2
/var/lib/docker/overlay2/l/LCTKPRHP6LG7KC7JQHETKIL6TZ
/var/lib/docker/overlay2/l/JOECSCSAQ5CPNHGEURVRT4JRQQ
upperdir=/var/lib/docker/overlay2/433751e2378f9b11.../diff
workdir=/var/lib/docker/overlay2/433751e2378f9b11.../work,xino=off)
```

`tr` 命令将冒号和逗号转换为换行符，以使输出更加易读。

`mount` 命令显示了 `lowerdir` 的七个单独条目，每个条目对应 NGINX 容器镜像中的一层。这七个目录，加上 `upperdir`，在 overlay 文件系统中合并在一起。

我们可以在挂载目录和上层目录中看到我们之前创建的 10MB 数据文件：

```
root@host01:~# ls -l $ROOT/tmp/data
-rw-r--r-- 1 root root 10485760 May 25 00:27 /var/lib/.../merged/tmp/data
root@host01:~# ls -l $ROOT/../diff/tmp/data
-rw-r--r-- 1 root root 10485760 May 25 00:27 /var/lib/.../diff/tmp/data
```

实际的文件存储在上层目录 *diff* 中，而挂载目录 *merged* 只是 overlay 文件系统生成的视图。

通常，我们不需要从宿主机深入容器文件系统，因为我们可以直接从容器内运行命令来探索其文件。然而，这种技术在容器引擎行为不正常时拉取容器文件会非常有用。

#### 实用的镜像构建建议

使用容器镜像时，overlay 文件系统的方式带来了一些重要的实际影响。首先，由于 overlay 文件系统可以有多个下层目录，且合并操作具有高效性，因此将容器镜像分为多个层几乎不会带来性能损失。这使得我们在构建容器镜像时能够非常模块化，方便复用各层。例如，我们可以从一个基础镜像开始，然后在其上构建一个安装了常用依赖的镜像，再构建另一个镜像，加入一些特定应用组件的依赖，最后构建另一个镜像，添加特定的应用程序。使用分层的方法组装应用容器镜像可以实现非常高效的镜像传输和存储，因为基础层可以在各个组件之间共享。

其次，因为在上层删除一个文件并不会真正删除下层的文件，我们需要小心如何处理大型临时文件，以及在构建镜像时如何存储机密信息。在这两种情况下，如果在文件仍然存在时完成了一层构建，它将永远存在，从而浪费带宽和空间，甚至更糟，泄露机密信息给下载镜像的任何人。一般来说，你应该假设每一行 *Dockerfile* 都会生成一个新的层，而且你应该假设每个命令相关的所有信息都会存储在镜像的元数据中。因此：

+   在一个 `RUN` 行中执行多个步骤，并确保每个 `RUN` 命令在执行后自行清理。

+   不要使用 `COPY` 命令将大文件或机密信息转移到镜像中，即使你在后续的 `RUN` 步骤中清理它们。

+   不要使用 `ENV` 存储机密信息，因为最终生成的值会成为镜像元数据的一部分。

### 开放容器倡议

一个容器镜像不仅仅是构成覆盖文件系统的层集合。它还包括重要的元数据，如容器的初始命令以及该命令的任何环境变量。开放容器倡议（OCI）提供了存储图像信息的标准格式。它确保由一个工具构建的容器镜像可以被任何其他工具使用，并提供了逐层或完整包裹传输图像的标准方式。

为了演示 OCI 格式，让我们从 Docker 中提取一个 BusyBox 容器镜像，并使用 Skopeo 将其存储为 OCI 格式，Skopeo 是一个设计用于在仓库和格式之间移动容器镜像的程序。第一步是提取镜像：

```
root@host01:~# skopeo copy docker-daemon:busybox:latest oci:busybox:latest
...
```

这条命令告诉 Skopeo 从 Docker 引擎的存储中获取镜像，并以 OCI 格式输出。现在我们有一个包含该镜像的 *busybox* 目录：

```
root@host01:~# ls -l busybox
total 12
drwxr-xr-x 3 root root 4096 May 24 23:59 blobs
-rw-r--r-- 1 root root  247 May 24 23:59 index.json
-rw-r--r-- 1 root root   31 May 24 23:59 oci-layout
```

*oci-layout* 文件指定了用于此镜像的 OCI 版本：

```
root@host01:~# jq . busybox/oci-layout
{
  "imageLayoutVersion": "1.0.0"
}
```

*index.json* 文件告诉我们有关该镜像的信息：

```
root@host01:~# jq . busybox/index.json
{
  "schemaVersion": 2,
 "manifests": [
    {
      "mediaType": "application/vnd.oci.image.manifest.v1+json",
      "digest": "sha256:9c3c5aeeaa7e1629871808339...",
      "size": 347,
      "annotations": {
        "org.opencontainers.image.ref.name": "latest"
      }
    }
  ]
}
```

*manifests* 属性是一个允许我们在单个 OCI 目录或包中存储多个镜像的数组。实际的文件系统内容按层存储在 *blobs* 目录中，每个层作为单独的 *.tar* 文件，因此任何共享层只存储一次。

这个 BusyBox 镜像只有一个单独的层。要查看其内容，我们需要通过 *index.json* 和镜像清单找到其 *.tar* 文件的路径：

```
root@host01:~# MANIFEST=$(jq -r \
  .manifests[0].digest busybox/index.json | sed -e 's/sha256://')
root@host01:~# LAYER=$(jq -r \
  .layers[0].digest busybox/blobs/sha256/$MANIFEST | sed -e 's/sha256://')
root@host01:~# echo $LAYER
197dfd3345530fd558a64f2a550e8af75a9cb812df5623daf0392aa39e0ce767
```

*blobs* 目录中的文件使用从文件内容计算得出的 SHA-256 摘要命名。我们首先使用 `jq` 获取 BusyBox 镜像清单的摘要，去掉前面的 `sha256:` 部分以获取清单文件的名称。然后读取清单以找到第一个（也是唯一的）层。现在我们可以看到这一层的内容：

```
root@host01:~# tar tvf busybox/blobs/sha256/$LAYER
drwxr-xr-x 0/0               0 2021-05-17 19:07 bin/
-rwxr-xr-x 0/0         1149184 2021-05-17 19:07 bin/
hrwxr-xr-x 0/0               0 2021-05-17 19:07 bin/[[ link to bin/[
...
drwxr-xr-x 0/0               0 2021-05-17 19:07 dev/
drwxr-xr-x 0/0               0 2021-05-17 19:07 etc/
...
```

将 `tar` 命令传递 `tvf` 告诉它列出我们指定的文件的内容表，这里是 BusyBox 镜像层。该层包含一个完整的 Linux 文件系统，其中 BusyBox 作为大多数标准 Linux 命令的单个可执行文件。

使用这个 *busybox* 目录，我们还可以将容器镜像打包起来，移动到另一个系统，然后在另一个容器引擎中拉取它。

### 总结思路

运行容器时，我们会得到一个看起来是独立的、隔离的文件系统，可以按需修改。在底层，容器引擎使用覆盖文件系统将多个容器镜像层合并到一起，并使用可写目录存储我们所做的所有更改。使用覆盖文件系统不仅使新容器快速启动，还意味着我们可以从同一镜像运行多个容器而无需等待文件复制完成，并且可以通过共享镜像层来减少所需的磁盘空间。

现在我们已经了解了进程隔离、资源限制、网络隔离和容器存储，这些都是容器在打包、分发、更新和运行应用组件时非常有价值的主要特性。接下来，我们将讨论只有在像 Kubernetes 这样的容器编排环境中才能获得的关键特性。我们将在[第二部分中进行讨论。
