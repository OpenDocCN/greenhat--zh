# 第十章：当事情出错时

![image](img/common01.jpg)

到目前为止，我们的 Kubernetes 安装和配置进展顺利，控制器在创建 Pods 和启动容器方面没有问题。当然，在现实世界中，事情很少这么简单。虽然无法展示复杂应用部署中可能出现的所有问题，但我们可以看看一些最常见的问题。最重要的是，我们可以探索一些调试工具，帮助我们诊断任何问题。

在本章中，我们将研究如何诊断在 Kubernetes 上部署的应用容器的问题。我们将循序渐进地了解调度和运行容器的生命周期，检查每个步骤可能出现的问题，以及如何诊断和解决它们。

### 调度

调度是 Kubernetes 对 Pod 及其容器执行的第一个操作。当一个 Pod 被创建时，Kubernetes 调度器会将其分配给一个节点。通常，这个过程会很快自动完成，但某些问题可能会阻止调度的成功执行。

#### 无可用节点

一种可能性是调度器根本没有可用的节点。这种情况可能是因为我们的集群没有配置任何用于常规应用容器的节点，或者因为所有节点都已失败。

为了说明没有可用节点进行分配的情况，我们将创建一个带有 *节点选择器* 的 Pod。节点选择器指定一个或多个节点标签，Pod 必须在匹配这些标签的节点上进行调度。节点选择器在集群中的某些节点与其他节点有所不同时很有用（例如，当一些节点拥有更新的 CPU，支持容器所需的更高级指令集时）。

**注意**

*本书的示例仓库位于* [`github.com/book-of-kubernetes/examples`](https://github.com/book-of-kubernetes/examples)。*有关设置的详细信息，请参见 第 xx 页 中的“运行示例”部分。*

我们将从一个具有节点选择器的 Pod 定义开始，这个选择器与我们的任何节点都不匹配：

*nginx-selector.yaml*

```
---
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - name: nginx
    image: nginx
  nodeSelector:
 ➊ purpose: special
```

节点选择器 ➊ 告诉 Kubernetes 只将这个 Pod 分配给一个标签为 `purpose` 且值为 `special` 的节点。尽管我们当前没有节点匹配该标签，我们仍然可以创建这个 Pod：

```
root@host01:~# kubectl apply -f /opt/nginx-selector.yaml
pod/nginx created
```

然而，Kubernetes 在尝试调度 Pod 时遇到了问题，因为它找不到匹配的节点：

```
root@host01:~# kubectl get pods -o wide
NAME    READY   STATUS    RESTARTS   AGE    IP       NODE     ...
nginx   0/1     Pending   0          113s   <none>   <none>   ...
```

我们看到的状态是 `Pending`，节点分配为 `<none>`。这是因为 Kubernetes 还没有将这个 Pod 调度到一个节点上。

`kubectl get` 命令通常是我们应该运行的第一个命令，用于查看我们部署到集群中的资源是否存在问题。如果出现问题，就像在本例中一样，下一步是使用 `kubectl describe` 查看详细的状态和事件日志：

```
root@host01:~# kubectl describe pod nginx
Name:         nginx
Namespace:    default
...
Status:       Pending
...
Node-Selectors:              purpose=special

Events:
  Type     Reason            Age    From               Message
  ----     ------            ----   ----               -------
  Warning  FailedScheduling  4m36s  default-scheduler  0/3 nodes are 
    available: 3 node(s) didn't match Pod's node affinity/selector.
  Warning  FailedScheduling  3m16s  default-scheduler  0/3 nodes are 
    available: 3 node(s) didn't match Pod's node affinity/selector.
```

事件日志告诉我们具体问题所在：Pod 无法调度，因为没有节点匹配选择器。

让我们向其中一个节点添加必要的标签：

```
root@host01:~# kubectl get nodes
NAME     STATUS   ROLES        ...
host01   Ready    control-plane...
host02   Ready    control-plane...
host03   Ready    control-plane...
root@host01:~# kubectl label nodes host02 purpose=special
node/host02 labeled
```

我们首先列出可用的三个节点，然后将必要的标签应用到其中一个节点上。一旦我们应用了这个标签，Kubernetes 现在可以调度该 Pod：

```
root@host01:~# kubectl get pods -o wide
NAME    READY   STATUS    RESTARTS   AGE   IP               NODE     ...
nginx   1/1     Running   0          10m   172.31.89.196   host02   ...
root@host01:~# kubectl describe pod nginx
Name:         nginx
Namespace:    default
...
Events:
  Type     Reason            Age    From               Message
  ----     ------            ----   ----               -------
  Warning  FailedScheduling  10m    default-scheduler  0/3 nodes are 
    available: 3 node(s) didn't match Pod's node affinity/selector.
 Warning  FailedScheduling  9m17s  default-scheduler  0/3 nodes are 
    available: 3 node(s) didn't match Pod's node affinity/selector.
  Normal   Scheduled         2m22s  default-scheduler  Successfully assigned 
    default/nginx to host02
...
```

正如预期的那样，Pod 被调度到了我们应用了标签的节点上。

这个示例，与本章中我们将看到的其他示例一样，展示了如何在 Kubernetes 中进行调试。在我们创建了所需的资源后，我们查询集群状态，以确保这些资源的实际部署成功。当我们发现问题时，可以纠正这些问题，我们的资源将按照预期启动，而无需重新安装我们的应用组件。

让我们清理一下这个 NGINX Pod：

```
root@host01:~# kubectl delete -f /opt/nginx-selector.yaml
pod "nginx" deleted
```

让我们也从节点中移除标签。我们通过在标签后添加一个减号来移除它，以便标识：

```
root@host01:~# kubectl label nodes host02 purpose-
node/host02 unlabeled
```

我们已经解决了一个关于调度器的问题，但还有另一个问题需要我们关注。

#### 资源不足

在选择节点来托管 Pod 时，调度器还会考虑每个节点上可用的资源以及 Pod 所需的资源。我们在第十四章中详细探讨了资源限制；目前只需要知道，每个容器都可以请求它所需的资源，调度器将确保它被调度到一个有这些资源可用的节点。当然，如果没有节点有足够的资源，调度器将无法调度该 Pod。相反，Pod 会处于 `Pending` 状态等待。

让我们看一个示例 Pod 定义来说明这一点：

*sleep-multiple.yaml*

```
---
apiVersion: v1
kind: Pod
metadata:
  name: sleep
spec:
  containers:
  - name: sleep
    image: busybox
 command: 
      - "/bin/sleep"
      - "3600"
    resources:
      requests:
        cpu: "2"
  - name: sleep2
    image: busybox
    command: 
      - "/bin/sleep"
      - "3600"
    resources:
      requests:
        cpu: "2"
```

在这个 YAML 定义中，我们在同一个 Pod 中创建了两个容器。每个容器请求两个 CPU。因为 Pod 中的所有容器必须在同一个主机上，以便共享某些 Linux 命名空间类型（尤其是网络命名空间，这样它们可以使用 `localhost` 进行通信），所以调度器需要找到一个有四个 CPU 可用的单一节点。在我们的一个小集群中，这是不可能的，正如我们尝试部署该 Pod 时所看到的那样：

```
root@host01:~# kubectl apply -f /opt/sleep-multiple.yaml
pod/sleep created
root@host01:~# kubectl get pods -o wide
NAME    READY   STATUS    RESTARTS   AGE   IP       NODE   ...
sleep   0/2     Pending   0          7s    <none>   <none> ...
```

如之前所述，`kubectl describe` 给出了事件日志，揭示了问题：

```
root@host01:~# kubectl describe pod sleep
Name:         sleep
Namespace:    default
...
Events:
  Type     Reason            Age   From               Message
  ----     ------            ----  ----               -------
  Warning  FailedScheduling  71s   default-scheduler  0/3 nodes are 
    available: 3 Insufficient cpu.
```

请注意，无论我们的节点实际负载有多重，都无关紧要：

```
root@host01:~# kubectl top node
NAME     CPU(cores)   CPU%   MEMORY(bytes)   MEMORY%   
host01   429m         21%    1307Mi          69%
host02   396m         19%    1252Mi          66%
host03   458m         22%    1277Mi          67%
```

容器实际使用多少 CPU 也无关紧要。调度器完全根据请求来分配 Pod；通过这种方式，当负载增加时，我们不会突然让 CPU 超负荷。

我们不能神奇地为我们的节点提供更多 CPU，因此，要让这个 Pod 被调度，我们需要为两个容器指定较低的 CPU 使用量。我们可以使用一个更合理的值：0.1 CPU：

*sleep-sensible.yaml*

```
---
apiVersion: v1
kind: Pod
metadata:
  name: sleep
spec:
  containers:
  - name: sleep
    image: busybox
    command: 
      - "/bin/sleep"
      - "3600"
    resources:
      requests:
     ➊ cpu: "100m"
  - name: sleep2
    image: busybox
    command: 
      - "/bin/sleep"
      - "3600"
    resources:
      requests:
        cpu: "100m"
```

值 `100m` ➊ 等同于“100 毫 CPU”或 CPU 的十分之一 (0.1)。

即使这是一个单独的文件，它声明了相同的资源，因此 Kubernetes 会将其视为更新。然而，如果我们尝试将其应用为对现有 Pod 的更改，它将失败：

```
root@host01:~# kubectl apply -f /opt/sleep-sensible.yaml
The Pod "sleep" is invalid: spec: Forbidden: pod updates may not change 
  fields other than ...
```

我们不允许更改已存在 Pod 的资源请求，这也合乎逻辑，因为 Pod 在创建时只会分配给节点一次，改变资源使用可能会导致节点过载。

如果我们使用的是像 Deployment 这样的控制器，控制器可以为我们处理替换 Pods 的操作。由于我们是直接创建 Pod，因此需要手动删除然后重新创建它：

```
root@host01:~# kubectl delete pod sleep
pod "sleep" deleted
root@host01:~# kubectl apply -f /opt/sleep-sensible.yaml
pod/sleep created
```

我们的新 Pod 在节点分配上没有问题：

```
root@host01:~# kubectl get pods -o wide
NAME    READY   STATUS    RESTARTS   AGE   IP               NODE  ...
sleep   2/2     Running   0          51s   172.31.89.199   host02 ...
```

如果我们在节点上运行 `kubectl describe`，可以看到我们的新 Pod 已经分配到节点的一些 CPU：

```
root@host01:~# kubectl describe node host02
Name:               host02
...
Capacity:
  cpu:                2
...
Non-terminated Pods:          (10 in total)
  Namespace  Name     CPU Requests  CPU Limits  ...
  ---------  ----     ------------  ----------  ...
...
  default    sleep ➊ 200m (10%)    0 (0%)      ... 
...
```

请确保使用正确的节点名称，指向部署 Pod 的节点。因为我们的 Pod 有两个容器，每个请求 `100m`，所以它的总请求为 `200m` ➊。

让我们最后清理这个 Pod：

```
root@host01:~# kubectl delete pod sleep
pod "sleep" deleted
```

其他错误可能会阻止 Pod 被调度，但这些是最常见的问题。最重要的是，我们在这里使用的命令适用于所有情况。首先，使用 `kubectl get` 来确定 Pod 的当前状态，然后使用 `kubectl describe` 查看事件日志。这两个命令在出现问题时总是一个不错的起点。

### 拉取镜像

Pod 被调度到节点上后，本地的 `kubelet` 服务会与底层的容器运行时交互，创建一个隔离的环境并启动容器。然而，仍然有一个应用配置错误可能导致我们的 Pod 停留在 `Pending` 阶段：无法拉取容器镜像。

三个主要问题可能会导致容器运行时无法拉取镜像：

+   无法连接到容器镜像注册表

+   请求的镜像授权问题

+   镜像在注册表中缺失

正如我们在 第五章 中所描述的，镜像注册表是一个 Web 服务器。通常，镜像注册表位于集群外部，节点需要能够连接到外部网络或互联网才能访问注册表。此外，大多数注册表支持发布需要身份验证和授权才能访问的私有镜像。当然，如果没有发布我们指定名称的镜像，容器运行时将无法从注册表拉取它。

所有这些错误在我们的 Kubernetes 集群中表现相同，仅在事件日志中的信息有所不同，因此我们只需要探索其中一个错误。我们将重点讨论可能最常见的问题：由于镜像名称中的拼写错误导致镜像缺失。

让我们尝试使用这个 YAML 文件创建一个 Pod：

*nginx-typo.yaml*

```
---
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - name: nginx
    image: nginz
```

因为在 Docker Hub 中没有名为 `nginz` 的镜像，所以无法拉取这个镜像。让我们看看将此资源添加到集群时会发生什么：

```
root@host01:~# kubectl apply -f /opt/nginx-typo.yaml
pod/nginx created
root@host01:~# kubectl get pods
NAME    READY   STATUS             RESTARTS   AGE
nginx   0/1     ImagePullBackOff   0          20s
```

我们的 Pod 状态是`ImagePullBackOff`，这立即传达了两个信息。首先，这个 Pod 尚未到达容器运行的阶段，因为它还没有拉取容器镜像。其次，与所有错误一样，Kubernetes 会继续尝试该操作，但会使用*退避*算法来避免让我们的集群资源过载。拉取镜像涉及通过网络与镜像仓库通信，如果在短时间内频繁发起请求，这对仓库来说既不礼貌也浪费网络带宽。此外，故障的原因可能是暂时性的，因此集群将继续尝试，希望问题能够解决。

Kubernetes 使用退避算法来重试错误，这对于调试非常重要。在这种情况下，我们显然不会将一个`nginx`镜像发布到 Docker Hub 来解决问题。但在一些我们通过发布镜像或更改镜像权限来修复问题的情况下，了解 Kubernetes 不会立即获取到这些更改也很重要，因为每次失败后，重试之间的延迟时间会增加。

让我们查看事件日志，以便看到这个退避过程的实际效果：

```
root@host01:~# kubectl describe pod nginx
Name:         nginx
Namespace:    default
...
Status:     ➊ Pending 
...
Events:
  Type     Reason     Age                 From               Message
  ----     ------     ----                ----               -------
  Normal   Scheduled  114s                default-scheduler  Successfully 
    assigned default/nginx to host03
...
  Warning  Failed     25s (x4 over 112s)  kubelet            Failed to pull 
    image "nginz": ... ➋ pull access denied, repository does not exist or may 
    require authorization  ...
...
  Normal   BackOff    1s ➌ (x7 over 111s)   kubelet            ...
```

如前所述，我们的 Pod 仍然处于`Pending`状态 ➊。然而，这时 Pod 已经完成了调度活动，并且开始拉取镜像。出于安全考虑，镜像仓库并不区分我们没有权限访问的私有镜像和缺失的镜像，因此 Kubernetes 只能告诉我们问题是两者之一 ➋。最后，我们可以看到 Kubernetes 在我们创建 Pod 的两分钟内尝试了七次拉取镜像 ➌，并且最后一次尝试是在一秒钟前。

如果我们等待几分钟，然后再次运行相同的`kubectl describe`命令，重点观察退避行为，我们可以看到每次重试之间的时间间隔变得非常长：

```
root@host01:~# kubectl describe pod nginx
Name:         nginx
Namespace:    default
...
Events:
  Type     Reason     Age                   From               Message
  ----     ------     ----                  ----               -------
...
  Normal   BackOff    4m38s (x65 over 19m)  kubelet            ...
```

现在 Kubernetes 已经在 19 分钟内尝试了 65 次拉取镜像。然而，随着时间的推移，延迟已经增加，且每次尝试之间的最大延迟达到了五分钟。这意味着在我们调试这个问题时，每次都需要等待最多五分钟来查看问题是否已解决。

让我们继续解决这个问题，以便能够看到实际效果。我们可以修复 YAML 文件并再次运行`kubectl apply`，但我们也可以使用`kubectl set`来修复它：

```
root@host01:~# kubectl set image pod nginx nginx=nginx
pod/nginx image updated
root@host01:~# kubectl get pods
NAME    READY   STATUS             RESTARTS   AGE
nginx   0/1     ImagePullBackOff   0          28m
```

`kubectl set`命令要求我们指定资源类型和名称；在本例中是`pod nginx`。然后我们指定`nginx=nginx`来提供要修改的容器名称（因为一个 Pod 可以有多个容器）以及新镜像。

我们修正了镜像名称，但 Pod 仍然显示`ImagePullBackOff`，因为我们必须等待五分钟的计时器结束，Kubernetes 才会再次尝试。下一次尝试时，镜像拉取成功，Pod 开始运行：

```
root@host01:~# kubectl get pods
NAME    READY   STATUS    RESTARTS   AGE
nginx   1/1     Running   0          32m
```

在继续之前，让我们先清理一下 Pod：

```
root@host01:~# kubectl delete pod nginx
pod "nginx" deleted
```

再次说明，我们通过使用 `kubectl get` 和 `kubectl describe` 解决了问题。然而，当容器运行起来时，这些命令就不足以提供帮助了。

### 运行中的容器

在指示容器运行时拉取所需镜像后，`kubelet` 会告诉运行时启动容器。对于本章中的其他示例，我们假设容器运行时按预期工作。此时，我们将面临的主要问题是容器未按预期启动。让我们从一个简单的调试示例开始，看看容器无法运行的情况，然后再看一个更复杂的示例。

#### 使用日志进行调试

对于我们的简单示例，我们首先需要一个 Pod 定义，里面的容器在启动时会失败。下面是一个会导致失败的 PostgreSQL Pod 定义：

*postgres-misconfig.yaml*

```
---
apiVersion: v1
kind: Pod
metadata:
  name: postgres
spec:
  containers:
  - name: postgres
    image: postgres
```

这个定义看起来似乎没有问题，但 PostgreSQL 在容器中运行时有一些必要的配置。

我们可以使用 `kubectl apply` 来创建 Pod：

```
root@host01:~# kubectl apply -f /opt/postgres-misconfig.yaml
pod/postgres created
```

等待大约一分钟以便拉取镜像后，我们可以使用 `kubectl get` 检查状态，这时我们会看到一个之前没有见过的状态：

```
root@host01:~# kubectl get pods
NAME       READY   STATUS             RESTARTS     AGE
postgres   0/1     CrashLoopBackOff   1 (8s ago)   25s
```

`CrashLoopBackOff` 状态表示 Pod 中的一个容器已经退出。由于这不是一个 Kubernetes 作业，它并不期望容器退出，所以它被认为是崩溃。

如果你在合适的时间查看 Pod，可能会看到 `Error` 状态，而不是 `CrashLoopBackOff`。这是暂时的：Pod 在崩溃后会立即过渡到该状态。

与 `ImagePullBackOff` 状态类似，`CrashLoopBackOff` 使用一种算法来重试失败，每次失败时增加重试之间的时间，以避免给集群带来过大负担。我们可以等待几分钟，再次打印状态来查看这种退避情况：

```
root@host01:~# kubectl get pods
NAME       READY   STATUS             RESTARTS       AGE
postgres   0/1     CrashLoopBackOff   5 (117s ago)   5m3s
```

在经过五次重启后，我们已经进入了每次重试之间需要等待超过一分钟的状态。等待时间将继续增加，直到达到五分钟，然后 Kubernetes 会继续每五分钟重试一次，直到无限期地进行下去。

让我们像往常一样使用 `kubectl describe` 尝试获取更多关于此失败的信息：

```
root@host01:~# kubectl describe pod postgres
Name:         postgres
Namespace:    default
...
Containers:
  postgres:
...
    State:          Waiting
      Reason:       CrashLoopBackOff
    Last State:     Terminated
      Reason:       Error
      Exit Code:    1
...
Events:
  Type     Reason     Age                    From               Message
  ----     ------     ----                   ----               -------
...
  Warning  BackOff    3m13s (x24 over 8m1s)  kubelet            Back-off 
    restarting failed container
```

`kubectl describe` 命令确实为我们提供了一个有用的信息：容器的退出代码。然而，这只是告诉我们发生了某种错误；它不足以完全调试失败的原因。为了弄清楚容器失败的原因，我们将使用 `kubectl logs` 命令查看容器日志：

```
root@host01:~# kubectl logs postgres
Error: Database is uninitialized and superuser password is not specified.
  You must specify POSTGRES_PASSWORD to a non-empty value for the
  superuser. For example, "-e POSTGRES_PASSWORD=password" on "docker run".
...
```

即使容器已经停止，我们依然可以查看日志，因为容器运行时已经捕获了它们。

这个消息直接来自 PostgreSQL 本身。幸运的是，它告诉我们问题的具体原因：我们缺少一个必需的环境变量。我们可以通过更新 YAML 资源文件快速修复这个问题：

*postgres-fixed.yaml*

```
---
apiVersion: v1
kind: Pod
metadata:
 name: postgres
spec:
  containers:
  - name: postgres
    image: postgres
 ➊ env:
    - name: POSTGRES_PASSWORD
      value: "supersecret"
```

`env` 字段 ➊ 添加了一个配置，用来传递所需的环境变量。当然，在实际系统中，我们不会将这些信息直接写在 YAML 文件中以明文方式存储。我们将在第十六章中讨论如何保护这种信息。

要应用这个更改，我们首先需要删除 Pod 定义，然后将新的资源配置应用到集群中：

```
root@host01:~# kubectl delete pod postgres
pod "postgres" deleted
root@host01:~# kubectl apply -f /opt/postgres-fixed.yaml
pod/postgres created
```

如前所述，如果我们使用的是控制器（如 Deployment），我们可以直接更新 Deployment，它会为我们处理删除旧 Pod 并创建新 Pod 的任务。

现在我们已经修复了配置，我们的 PostgreSQL 容器按预期启动：

```
root@host01:~# kubectl get pods
NAME       READY   STATUS    RESTARTS   AGE
postgres   1/1     Running   0          77s
```

在继续下一个例子之前，让我们清理一下这个 Pod：

```
root@host01:~# kubectl delete pod postgres
pod "postgres" deleted
```

大多数编写良好的应用程序在终止之前会打印日志消息，但我们需要为更困难的情况做好准备。让我们再看一个包含两种新调试方法的例子。

#### 使用 Exec 调试

对于这个例子，我们需要一个表现不好的应用程序。我们将使用一个进行不当内存访问的 C 程序。这个程序被打包进一个 Alpine Linux 容器中，以便我们可以在 Kubernetes 中将其作为容器运行。以下是 C 源代码：

*crasher.c*

```
int main() {
  char *s = "12";
  s[2] = '3';
 return 0;
}
```

代码的第一行创建了一个指向长度为两个字符的字符串的指针；第二行尝试写入不存在的第三个字符，导致程序立即终止。

这个 C 程序可以通过使用`gcc`编译在任何系统上生成一个`crasher`可执行文件。如果你在主机 Linux 系统上构建，可以使用这个`gcc`命令：

```
$ gcc -g -static -o crasher crasher.c
```

`-g` 参数确保调试符号可用。我们稍后将使用这些符号。`-static` 参数最为重要；我们希望将其打包为一个独立的应用程序，放入 Alpine 容器镜像中。如果我们在其他 Linux 发行版（如 Ubuntu）上构建，标准库基于不同的工具链，动态链接将失败。因此，我们希望我们的可执行文件将所有依赖项静态链接。最后，我们使用`-o`来指定输出的可执行文件名称，并提供 C 源文件的名称。

另外，你可以直接使用已经构建并发布到 Docker Hub 的容器镜像，镜像名称为`bookofkubernetes/crasher: stable`。这个镜像是通过 GitHub Actions 自动构建并发布的，基于仓库中的代码 *[`github.com/book-of-kubernetes/crasher`](https://github.com/book-of-kubernetes/crasher)*。以下是该仓库中的 *Dockerfile*：

*Dockerfile*

```
FROM alpine AS builder
COPY ./crasher.c /
RUN apk --update add gcc musl-dev && \
    gcc -g -o crasher crasher.c

FROM alpine
COPY --from=builder /crasher /crasher
CMD [ "/crasher" ]
```

这个 *Dockerfile* 利用了 Docker 的多阶段构建功能，减少了最终镜像的大小。为了在 Alpine 容器中进行编译，我们需要`gcc`和核心的 C 头文件及库。然而，这些会使容器镜像显著增大。我们只在编译时需要它们，因此希望避免将这些额外内容包含在最终镜像中。

当我们使用在第五章中看到的 `docker build` 命令来构建时，Docker 会基于 Alpine Linux 创建一个容器，将我们的源代码复制到其中，安装开发工具，并编译应用程序。然后，Docker 会使用一个新的 Alpine Linux 容器重新开始，并将第一个容器中生成的可执行文件复制到新的容器中。最终的容器镜像来自第二个容器，因此我们避免将开发工具添加到最终镜像中。

让我们在 Kubernetes 集群中运行这个镜像。这次我们将使用 Deployment 资源，以便可以演示如何编辑它来解决崩溃的容器问题：

*crasher-deploy.yaml*

```
---
kind: Deployment
apiVersion: apps/v1
metadata:
  name: crasher
spec:
  replicas: 1
  selector:
    matchLabels:
      app: crasher
  template:
    metadata:
      labels:
        app: crasher
    spec:
      containers:
      - name: crasher
        image: bookofkubernetes/crasher:stable
```

这个基本的 Deployment 与我们在第七章中介绍的 Deployment 非常相似。我们指定 `image` 字段来匹配镜像发布的位置。

我们可以像往常一样将这个 Deployment 添加到集群中：

```
root@host01:~# kubectl apply -f /opt/crasher-deploy.yaml
deployment.apps/crasher created
```

一旦 Kubernetes 有机会调度 Pod 并拉取镜像，它就会开始崩溃，正如预期的那样：

```
root@host01:~# kubectl get pods
NAME                       READY   STATUS             RESTARTS      AGE
crasher-76cdd9f769-5blbn   0/1     CrashLoopBackOff   3 (24s ago)   73s
```

如前所述，使用 `kubectl describe` 只能告诉我们容器的退出代码。还有另一种获取退出代码的方法；我们可以使用 `kubectl get` 的 JSON 输出格式和 `jq` 工具来捕获退出代码：

```
root@host01:~# kubectl get pod crasher-7978d9bcfb-wvx6q -o json | \
  jq '.status.containerStatuses[].lastState.terminated.exitCode'
139
```

一定要使用 `kubectl get pods` 输出中的正确 Pod 名称。我们需要的特定字段路径是基于 Kubernetes 内部如何跟踪该资源的；通过一些实践，构建路径并使用 `jq` 捕获特定字段会变得更加容易，这是在脚本编写中非常有用的技巧。

退出代码 139 告诉我们容器因段错误而终止。然而，日志对于诊断问题没有帮助，因为我们的程序在崩溃之前没有打印任何信息：

```
root@host01:~# kubectl logs crasher-76cdd9f769-5blbn
[ no output ]
```

我们遇到了一个大问题。日志没有帮助，所以下一步是使用 `kubectl exec` 进入容器。然而，容器在应用程序崩溃后立即停止，没能维持足够长的时间让我们进行调试工作。

为了解决这个问题，我们需要一种方法来启动容器而不运行崩溃的程序。我们可以通过覆盖默认命令来让容器保持运行状态。由于我们是基于 Alpine Linux 镜像构建的，`sleep` 命令可供我们使用。

我们可以编辑 YAML 文件并更新 Deployment，但也可以直接使用 `kubectl edit` 命令编辑 Deployment，这样当前的定义会在编辑器中打开，我们所做的任何更改都会保存到集群中：

```
root@host01:~# kubectl edit deployment crasher
```

这将会在编辑器窗口中打开 vi，里面包含以 YAML 格式表示的 Deployment 资源。该资源会包含比我们创建时更多的字段，因为 Kubernetes 会向我们展示资源的状态以及一些带有默认值的字段。

如果你不喜欢 vi，可以在 `kubectl edit` 命令前加上 `KUBE_EDITOR=nano` 来使用 Nano 编辑器。

在文件中，找到这些行：

```
    spec:
      containers:
      - image: bookofkubernetes/crasher:stable
        imagePullPolicy: IfNotPresent
```

即使`imagePullPolicy`行未出现在 YAML 资源中，你仍会看到它，因为 Kubernetes 已自动将默认策略添加到资源中。在`image`和`imagePullPolicy`之间添加一行，使结果如下所示：

```
    spec:
      containers:
      - image: bookofkubernetes/crasher:stable
        args: ["/bin/sleep", "infinity"]
        imagePullPolicy: IfNotPresent
```

这行新增的代码覆盖了容器的默认命令，使其运行`sleep`，而不是运行我们的崩溃程序。保存并退出编辑器，`kubectl`将会加载新的定义：

```
deployment.apps/crasher edited
```

在`kubectl`将此更改应用到集群后，Deployment 必须删除旧的 Pod 并创建一个新的 Pod。这个过程会自动完成，因此我们唯一能注意到的区别是 Pod 名称中自动生成的部分。当然，我们还会看到 Pod 正在运行：

```
root@host01:~# kubectl get pods
NAME                       READY   STATUS    RESTARTS   AGE
crasher-58d56fc5df-vghbt   1/1     Running   0          3m29s
```

我们的 Pod 现在正在运行，但它只是在运行`sleep`。我们仍然需要调试我们的实际应用程序。为此，我们现在可以在容器内获取一个 Shell 提示符：

```
root@host01:~# kubectl exec -ti crasher-58d56fc5df-vghbt -- /bin/sh
/ #
```

当我们更改定义时，Deployment 替换了 Pod，因此名称发生了变化。如前所述，请使用正确的 Pod 名称。此时我们可以手动尝试我们的崩溃程序：

```
/ # /crasher
Segmentation fault (core dumped)
```

在许多情况下，能够以这种方式运行程序，调整不同的环境变量和命令行选项，可能足以找到并修复问题。或者，我们可以尝试使用`strace`来运行程序，它会告诉我们程序在崩溃之前尝试进行哪些系统调用以及尝试打开哪些文件。在这种情况下，我们知道程序因段错误崩溃，这意味着问题很可能是编程错误，因此我们最好的方法是通过端口转发将调试工具连接到应用程序。

#### 使用端口转发进行调试

我们将使用基于文本的调试器`gdb`来演示端口转发，但任何可以通过网络端口连接的调试器都可以使用。首先，我们需要使用一个调试器在容器内创建我们的应用程序，该调试器将在网络端口上监听，并在运行代码之前等待。为此，我们需要在容器内安装`gdb`。由于这是一个 Alpine 容器，我们将使用`apk`：

```
/ # apk add gdb
...
(13/13) Installing gdb (10.1-r0)
Executing busybox-1.32.1-r3.trigger
OK: 63 MiB in 27 packages
```

我们安装的`gdb`版本包含`gdbserver`，它使我们能够启动一个网络调试会话。

由于`gdb`是一个基于文本的调试器，我们显然可以直接启动它来调试应用程序，但使用带有 GUI 的调试器通常更为方便，因为它使我们更容易逐步调试源代码、设置断点和观察变量。因此，我将展示如何通过网络连接调试器的过程。

让我们启动`gdbserver`并设置它监听端口`2345`：

```
/ # gdbserver localhost:2345 /crasher
Process /crasher created; pid = 25
Listening on port 2345
```

请注意，我们告诉`gdbserver`监听`localhost`接口。我们仍然可以连接到调试器，因为 Kubernetes 将为我们提供通过`kubectl port-forward`命令进行端口转发的功能。此命令使`kubectl`连接到 API 服务器，并请求它将流量转发到指定 Pod 上的特定端口。其优势在于，我们可以从任何能够连接到 API 服务器的地方使用此端口转发功能，甚至是集群外部。

专门使用端口转发来运行远程调试器可能不是 Kubernetes 集群管理员或容器化应用程序开发人员的日常工作，但当没有其他方法找到 bug 时，这是一项有价值的技能。它也是展示端口转发功能以访问 Pod 的一个绝佳方式。

由于我们的调试器正在第一个终端中运行，我们需要另一个终端标签或窗口来进行端口转发，这可以从我们集群中的任何主机完成。我们使用`host01`：

```
root@host01:~# kubectl port-forward pods/crasher-58d56fc5df-vghbt 2345:2345
Forwarding from 127.0.0.1:2345 -> 2345
Forwarding from [::1]:2345 -> 2345
```

该`kubectl`命令开始在端口`2345`上监听，并将所有流量通过 API 服务器转发到我们指定的 Pod。由于此命令会持续运行，我们需要另一个终端窗口或标签来执行我们的最后一步，即运行将用于连接到容器中运行的调试服务器的调试器。这必须在与`kubectl port-forward`命令相同的主机上完成，因为该程序仅在本地接口上监听。

此时，我们可以运行任何知道如何与调试服务器通信的调试器。为了简单起见，我们再次使用`gdb`。我们将首先切换到*/opt*目录，因为我们的 C 源文件在那里：

```
root@host01:~# cd /opt
```

现在我们可以启动`gdb`并使用它连接到调试服务器：

```
root@host01:/opt# gdb -q
(gdb) target remote localhost:2345
Remote debugging using localhost:2345
...
Reading /crasher from remote target...
Reading symbols from target:/crasher...
0x0000000000401bc0 in _start ()
```

我们的调试会话成功连接并等待我们启动程序，我们将使用`continue`命令来启动：

```
(gdb) continue
Continuing.

Program received signal SIGSEGV, Segmentation fault.
main () at crasher.c:3
3         s[2] = '3';
```

使用调试器，我们能够准确看到哪一行源代码导致了段错误，现在我们可以弄清楚如何修复它。

### 最终想法

当我们将应用程序组件移入容器镜像并在 Kubernetes 集群中运行时，我们在可扩展性和自动故障转移方面获得了巨大的好处，但也引入了在启动应用程序时可能出错的新问题，并带来了调试这些问题的新挑战。在本章中，我们探讨了如何使用 Kubernetes 命令系统地跟踪我们的应用程序启动和运行，以确定是什么阻止了它正常工作。通过这些命令，我们可以调试在应用程序级别发生的任何问题，即使某个应用程序组件在容器化环境中无法正确启动。

现在我们已经清楚地了解了如何使用 Kubernetes 运行容器，接下来我们可以深入研究集群本身的功能。在这个过程中，我们将确保探讨每个组件的工作原理，以便拥有诊断问题所需的工具。我们将在下一章开始，详细了解 Kubernetes 控制平面。
