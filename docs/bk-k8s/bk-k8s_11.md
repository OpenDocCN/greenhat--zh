# 第九章：服务和入口网络

![image](img/common01.jpg)

创建一个集群级别的网络，使得所有 Pod 可以互相通信，涉及了相当复杂的操作。同时，我们仍然没有获得构建可扩展、弹性应用所需的所有网络功能。我们需要支持将应用组件跨多个实例进行负载均衡的网络，并且提供将流量发送到新的 Pod 实例的能力，以应对现有实例的故障或升级需求。此外，Pod 网络设计为私有网络，意味着它仅能从集群内部直接访问。我们需要额外的流量路由功能，以便外部用户可以访问我们在容器中运行的应用组件。

在本章中，我们将讨论服务和入口网络。Kubernetes 的服务网络提供了一个额外的网络层，位于 Pod 网络之上，包括动态发现和负载均衡。我们将看到这个网络层如何工作，以及如何利用它将我们的应用组件暴露给集群中的其他部分，作为可扩展和有弹性的服务。然后，我们将探讨入口配置如何为这些服务提供流量路由，将它们暴露给外部用户。

### 服务

将部署和覆盖网络结合起来，我们可以创建多个相同的容器实例，每个实例都有一个唯一的 IP 地址。让我们创建一个 NGINX 部署来说明：

*nginx-deploy.yaml*

```
---
kind: Deployment
apiVersion: apps/v1
metadata:
  name: nginx
spec:
  replicas: 5
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx
```

这与我们之前看到的部署类似。在这种情况下，我们要求 Kubernetes 为我们维护五个 Pod，每个 Pod 运行一个 NGINX web 服务器。

**注意**

*本书的示例仓库位于* [`github.com/book-of-kubernetes/examples`](https://github.com/book-of-kubernetes/examples)。*有关设置的详细信息，请参见“运行示例”部分，第 xx 页。*

自动化脚本已经将此文件放置在*/opt*目录下，因此我们可以将其应用到集群中：

```
root@host01:~# kubectl apply -f /opt/nginx-deploy.yaml
deployment.apps/nginx created
```

在这些 Pod 启动后，我们可以检查它们是否已分布到集群中，并且每个 Pod 都有一个 IP 地址：

```
root@host01:~# kubectl get pods -o wide
NAME                     READY   STATUS    ... IP                NODE    ...
nginx-6799fc88d8-2wqc7   1/1     Running   ... 172.31.239.231   host01   ...
nginx-6799fc88d8-78bwx   1/1     Running   ... 172.31.239.229   host01   ...
nginx-6799fc88d8-dtx7s   1/1     Running   ... 172.31.89.240    host02   ...
nginx-6799fc88d8-wh479   1/1     Running   ... 172.31.239.230   host01   ...
nginx-6799fc88d8-zwx27   1/1     Running   ... 172.31.239.228   host01   ...
```

如果这些容器只是某个服务器的客户端，那可能就是我们需要做的全部了。例如，如果我们的应用架构是通过发送和接收消息来驱动的，只要这些容器能够连接到消息服务器，它们就能按要求工作。然而，因为这些容器充当服务器的角色，客户端需要能够找到它们并建立连接。

就目前而言，我们的独立 NGINX 实例对客户端来说并不太实用。当然，直接连接到这些 NGINX 服务器 Pod 中的任何一个都是可能的。例如，我们可以通过其 IP 地址与列表中的第一个进行通信：

```
root@host01:~# curl -v http://172.31.239.231
*   Trying 172.31.239.231:80...
* Connected to 172.31.239.231 (172.31.239.231) port 80 (#0)
> GET / HTTP/1.1
...
< HTTP/1.1 200 OK
< Server: nginx/1.21.3
...
```

不幸的是，单独选择一个实例并不能提供负载均衡或故障转移功能。此外，我们无法提前知道 Pod 的 IP 地址，而且每次对 Deployment 进行更改时，Pods 会被重新创建并获得新的 IP 地址。

解决这种情况需要具备两个主要特性。首先，我们需要一个客户端可以用来查找服务器的众所周知的名称。其次，我们需要一个一致的 IP 地址，这样当客户端识别到一个服务器时，即使 Pod 实例来来去去，也可以继续使用相同的地址进行连接。这正是 Kubernetes 通过 *Service* 提供的功能。

#### 创建 Service

让我们为我们的 NGINX Deployment 创建一个 Service，看看这能带来什么。清单 9-1 提供了资源的 YAML 文件。

*nginx-service.yaml*

```
---
kind: Service
apiVersion: v1
metadata:
  name: nginx
spec:
  selector:
    app: nginx
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
```

*清单 9-1：NGINX Service*

首先，Service 具有一个 `selector`，与 Deployment 类似。这个选择器以相同的方式使用：用来识别将与 Service 关联的 Pods。然而，与 Deployment 不同，Service 不以任何方式管理它的 Pods；它只是将流量路由到它们。

流量路由是基于我们在 `ports` 字段中指定的端口。由于 NGINX 服务器监听的是端口 80，我们需要将其指定为 `targetPort`。我们可以使用任何我们想要的 `port`，但最简单的做法是保持一致，特别是因为 80 是 HTTP 的默认端口。

让我们将这个 Service 应用到集群中：

```
root@host01:~# kubectl apply -f /opt/nginx-service.yaml 
service/nginx created
```

我们现在可以看到已经创建了 Service：

```
root@host01:~# kubectl get services
NAME         TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)   AGE
kubernetes   ClusterIP   10.96.0.1        <none>        443/TCP   14d
nginx        ClusterIP   10.100.221.220   <none>        80/TCP    25s
```

这个 `nginx` Service 默认类型为 `ClusterIP`。Kubernetes 已经为该 Service 自动分配了一个集群 IP 地址。该 IP 地址与我们的 Pods 的地址空间完全不同。

使用选择器，这个 Service 会识别我们的 NGINX 服务器 Pods，并自动开始将流量负载均衡到它们。当匹配选择器的 Pods 来来去去时，Service 会自动更新其负载均衡。只要 Service 存在，它就会保持相同的 IP 地址，这样客户端就能持续通过一致的方式找到我们的 NGINX 服务器实例。

让我们验证是否能够通过 Service 访问 NGINX 服务器：

```
root@host01:~# curl -v http://10.100.221.220
*   Trying 10.100.221.220:80...
* Connected to 10.100.221.220 (10.100.221.220) port 80 (#0)
> GET / HTTP/1.1
...
< HTTP/1.1 200 OK
< Server: nginx/1.21.3
...
```

我们可以看到，Service 已经正确地识别了所有五个 NGINX Pods：

```
root@host01:~# kubectl describe service nginx
Name:              nginx
Namespace:         default
...
Selector:          app=nginx
...
Endpoints:         172.31.239.228:80,172.31.239.229:80,172.31.239.230:80 
+ 2 more...
...
```

`Endpoints` 字段显示 Service 当前正在将流量路由到所有五个 NGINX Pods。作为客户端，我们不需要知道是哪个 Pod 处理了我们的请求。我们只与 Service IP 地址交互，允许 Service 为我们选择一个实例。

当然，在这个示例中，我们必须查找 Service 的 IP 地址。为了方便客户端，我们仍然应该提供一个众所周知的名称。

#### Service DNS

Kubernetes 通过 DNS（域名系统）服务器为每个服务提供一个众所周知的名称，该服务器会动态更新集群中每个服务的名称和 IP 地址。每个 Pod 都配置了这个 DNS 服务器，这样 Pod 就可以使用服务的名称来连接到一个实例。

让我们创建一个 Pod，以便我们可以尝试这个操作：

*pod.yaml*

```
---
apiVersion: v1
kind: Pod
metadata:
  name: pod
spec:
  containers:
  - name: pod
    image: alpine
    command: 
      - "sleep"
      - "infinity"
```

我们使用`alpine`而不是`busybox`作为这个 Pod 的镜像，因为我们需要使用一些 DNS 命令，这些命令要求我们安装一个功能更强大的 DNS 客户端。

**注意**

*BusyBox 是一个非常适合 Kubernetes 集群调试的镜像，因为它非常小并且包含许多有用的命令。然而，为了保持 BusyBox 的小巧，通常这些命令只包含最常用的选项。Alpine 是一个非常好的调试替代品。默认的 Alpine 镜像使用 BusyBox 来提供其许多初始命令，但通过安装适当的软件包，也可以将其替换为功能更完整的替代品。*

接下来，创建 Pod：

```
root@host01:~# kubectl apply -f /opt/pod.yaml 
pod/pod created
```

它启动后，让我们使用它连接到我们的 NGINX 服务，如示例 9-2 中所示。

```
root@host01:~# kubectl exec -ti pod -- wget -O - http://nginx
Connecting to nginx (10.100.221.220:80)
...
<title>Welcome to nginx!</title>
...
```

*示例 9-2：连接到 NGINX 服务*

我们能够使用服务的名称`nginx`，并且该名称解析为服务的 IP 地址。之所以能这样工作，是因为我们的 Pod 已经配置为与集群内置的 DNS 服务器通信：

```
root@host01:~# kubectl exec -ti pod -- cat /etc/resolv.conf 
search default.svc.cluster.local svc.cluster.local cluster.local 
nameserver 10.96.0.10
options ndots:5
```

我们打印容器内的文件*/etc/resolv.conf*，因为这是用来配置 DNS 的文件。

引用的名称服务器`10.96.0.10`本身就是一个 Kubernetes 服务，但它位于`kube-system`命名空间中，因此我们需要在该命名空间中查找它：

```
root@host01:~# kubectl -n kube-system get services
NAME            TYPE       CLUSTER-IP      ... PORT(S)                  AGE
kube-dns        ClusterIP  10.96.0.10      ... 53/UDP,53/TCP,9153/TCP   14d
metrics-server  ClusterIP  10.105.140.176  ... 443/TCP                  14d
```

`kube-dns`服务连接到一个名为 CoreDNS 的 DNS 服务器部署，该服务器监听 Kubernetes 集群中服务的变化。CoreDNS 根据需要更新 DNS 服务器配置，以保持与当前集群配置同步。

#### 名称解析和命名空间

Kubernetes 集群中的 DNS 名称是基于命名空间以及集群域的。由于我们的 Pod 位于`default`命名空间，因此它的搜索路径已被配置为`default.svc.cluster.local`，这是列表中的第一个条目，因此在查找服务时，它将首先搜索`default`命名空间。这就是为什么我们能够使用裸服务名称`nginx`来找到`nginx`服务的原因——该服务也位于`default`命名空间中。

我们也可以使用完全限定的名称找到相同的服务：

```
root@host01:~# kubectl exec -ti pod -- wget -O - http://nginx.default.svc
Connecting to nginx.default.svc (10.100.221.220:80)
...
<title>Welcome to nginx!</title>
...
```

理解命名空间和服务查找之间的相互作用非常重要。Kubernetes 集群的一个常见部署模式是将同一个应用程序多次部署到不同的命名空间，并使用简单的主机名使应用程序组件相互通信。这个模式通常用于将应用程序的“开发”版本和“生产”版本部署到同一个集群中。如果我们打算使用这种模式，我们需要确保在应用程序组件尝试相互发现时，坚持使用纯粹的主机名；否则，我们可能会与应用程序的错误版本进行通信。

在*/etc/resolv.conf*中另一个重要的配置项是`ndots`条目。`ndots`条目告诉主机名解析器，当它看到一个包含四个或更少点的主机名时，它应该先尝试附加各种搜索域，而不是在没有附加任何域名的情况下直接执行绝对查找。这对于确保我们在访问集群外部之前，尝试查找集群内的服务至关重要。

结果是，当我们在清单 9-2 中使用名称`nginx`时，我们容器内的 DNS 解析器立即尝试了`nginx.default.svc.cluster.local`并找到了正确的服务。

为了确保这一点清晰明了，我们再看一个例子：查找另一个命名空间中的服务。`kube-system`命名空间中有一个`metrics-server`服务。为了查找它，我们可以在 Pod 中使用标准的主机查找命令`dig`。

我们的 Pod 使用的是 Alpine Linux，因此我们需要安装`bind-tools`包来获取`dig`工具：

```
root@host01:~# kubectl exec -ti pod -- apk add bind-tools
...
OK: 13 MiB in 27 packages
```

现在，让我们首先使用纯主机名尝试查找`metrics-server`：

```
root@host01:~# kubectl exec -ti pod -- dig +search metrics-server
...
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: SERVFAIL, id: 38423
...
```

我们在命令中添加了`+search`标志，告诉`dig`使用来自*/etc/resolv.conf*的搜索路径信息。然而，即便有了这个标志，我们仍然无法找到服务，因为我们的 Pod 位于`default`命名空间，因此搜索路径没有让`dig`去查找`kube-system`命名空间。

让我们再试一次，这次指定正确的命名空间：

```
root@host01:~# kubectl exec -ti pod -- dig +search metrics-server.kube-system
...
;; ANSWER SECTION:
metrics-server.kube-system.svc.cluster.local. 30 IN A 10.105.140.176
...
```

这个查找成功了，我们能够获得`metrics-server`服务的 IP 地址。之所以成功，是因为搜索路径的第二个条目包括了`svc.cluster.local`。在最初尝试了`metrics-server.kube-system.default.svc.cluster.local`（失败）之后，`dig`接着尝试了`metrics-server.kube-system.svc.cluster.local`，这次成功了。

#### 流量路由

我们已经看到了如何创建和使用服务，但还没有了解实际的流量路由是如何工作的。事实证明，服务的网络流量工作方式与我们在第八章中看到的覆盖网络完全不同，这可能会导致一些混淆。

例如，虽然我们可以使用`wget`通过`nginx`服务名访问 NGINX 服务器实例，但我们可能会期待同样能够使用`ping`，然而这并不起作用：

```
root@host01:~# kubectl exec -ti pod -- ping -c 3 nginx
PING nginx (10.100.221.220): 56 data bytes

--- nginx ping statistics ---
3 packets transmitted, 0 packets received, 100% packet loss
command terminated with exit code 1
```

名称解析按预期工作，因此 `ping` 知道应该使用哪个目标 IP 地址来发送 ICMP 包。但是，从该 IP 地址没有收到回应。我们可以查看集群中每个主机和容器的网络接口，却永远找不到承载 `10.100.221.220` 这个 Service IP 地址的接口。那么，为什么我们的 HTTP 流量能够顺利到达 NGINX Service 实例呢？

在我们集群的每个节点上，都有一个名为 `kube-proxy` 的组件，它配置 Service 的流量路由。`kube-proxy` 作为一个 DaemonSet 在 `kube-system` 命名空间中运行。每个 `kube-proxy` 实例都会监视集群中 Service 的变化，并配置 Linux 防火墙以路由流量。

我们可以使用 `iptables` 命令查看防火墙配置，看看 `kube-proxy` 如何为我们的 `nginx` Service 配置流量路由：

```
   root@host01:~# iptables-save | grep 'default/nginx cluster IP'
➊ -A KUBE-SERVICES ! -s 172.31.0.0/16 -d 10.100.221.220/32 -p tcp -m comment 
    --comment "default/nginx cluster IP" -m tcp --dport 80 -j KUBE-MARK-MASQ
➋ -A KUBE-SERVICES -d 10.100.221.220/32 -p tcp -m comment --comment 
   "default/nginx cluster IP" -m tcp --dport 80 -j KUBE-SVC-2CMXP7HKUVJN7L6M
```

`iptables-save` 命令备份当前所有 Linux 防火墙规则，因此它对于打印所有规则非常有用。`grep` 命令用于搜索 `kube-proxy` 应用于它所创建的 Service 规则的注释字符串。在这个示例中，`kube-proxy` 为整个 Service 创建了两条规则。第一条规则 ➊ 查找目标是我们的 Service 且*不是*来自 Pod 网络的流量。这个流量必须标记为网络地址转换（NAT）伪装，以便任何响应流量的源地址会被重写为 Service IP 地址，而不是实际处理请求的 Pod。第二条规则 ➋ 将所有目标是 Service 的流量发送到一个独立的规则链，这个规则链会将流量转发到一个 Pod 实例。注意，在这两种情况下，规则只会匹配目标端口为 80 的 TCP 流量。

我们可以检查这个独立的规则链，看看实际是如何将流量路由到各个 Pod 实例的。确保在此命令中替换规则链的名称为之前输出中显示的名称：

```
root@host01:~# iptables-save | grep KUBE-SVC-2CMXP7HKUVJN7L6M
...
-A KUBE-SVC-2CMXP7HKUVJN7L6M ... -m statistic --mode random 
  --probability 0.20000000019 -j KUBE-SEP-PIVU7ZHMCSOWIZ2Z
-A KUBE-SVC-2CMXP7HKUVJN7L6M ... -m statistic --mode random 
  --probability 0.25000000000 -j KUBE-SEP-CFQXKE74QEHFB7VJ
-A KUBE-SVC-2CMXP7HKUVJN7L6M ... -m statistic --mode random 
  --probability 0.33333333349 -j KUBE-SEP-DHDWEJZ7MGGIR5XF
-A KUBE-SVC-2CMXP7HKUVJN7L6M ... -m statistic --mode random 
  --probability 0.50000000000 -j KUBE-SEP-3S3S2VJCXSAISE2Z
-A KUBE-SVC-2CMXP7HKUVJN7L6M ... -j KUBE-SEP-AQWD2Y25T24EHSNI
```

输出显示了五条规则，分别对应于 Service 的选择器匹配的五个 NGINX Pod 实例。这五条规则共同提供了跨所有实例的随机负载均衡，确保每个实例都有相同的机会被选中处理新的连接。

可能看起来有些奇怪的是，每条规则的 `probability` 数值会递增。这是必要的，因为规则是顺序评估的。对于第一条规则，我们希望有 20% 的概率选择第一个实例。然而，如果我们没有选择第一个实例，剩下的只有四个实例，因此我们希望有 25% 的概率选择第二个实例。相同的逻辑适用于所有后续实例，直到最后一个实例，我们希望在跳过了其他所有实例之后总是选择它。

让我们快速验证这些规则是否到达预期的目标地点（再次提醒，请确保在此命令中替换规则链的名称）：

```
root@host01:~# iptables-save | grep KUBE-SEP-PIVU7ZHMCSOWIZ2Z
...
-A KUBE-SEP-PIVU7ZHMC ... -s 172.31.239.235/32 ... --comment "default/nginx" -j KUBE-MARK-MASQ
-A KUBE-SEP-PIVU7ZHMCSOWIZ2Z -p tcp ... -m tcp -j DNAT --to-destination 172.31.239.235:80
```

该输出展示了两条规则。第一条是 NAT 伪装配置的另一半，我们标记了所有离开 Pod 实例的包，以便它们的源地址可以被重写，看起来是来自 Service。第二条规则实际上是将流量路由到特定 Pod 的规则，它执行目标地址的重写，使得原本应该发送到 Service IP 的包现在发送到 Pod。之后，覆盖网络接管，实际上将数据包发送到正确的容器。

通过理解 Service 流量是如何被路由的，我们可以理解为什么 ICMP 包没有通过。`kube-proxy` 创建的防火墙规则仅适用于目标端口为 80 的 TCP 流量。因此，没有防火墙规则来重写我们的 ICMP 包，因此它们无法到达能够回复它们的网络栈。类似地，如果我们有一个监听多个端口的容器，我们将能够直接使用 Pod 的 IP 地址连接到这些端口，但 Service IP 地址只会在我们明确声明该端口的 Service 规范时路由流量。这在部署应用程序时可能会引起混淆，Pod 按预期启动并监听流量，但 Service 配置错误导致流量无法路由到所有正确的目标端口。

### 外部网络

现在，我们已经具备了足够的网络层来满足所有内部集群通信需求。每个 Pod 都有自己的 IP 地址，并且可以连接到其他 Pod 以及控制平面，利用 Service 网络我们可以实现基于运行多个 Pod 实例的负载均衡和故障转移。然而，我们仍然缺少外部用户访问我们集群中服务的能力。

为了提供外部用户的访问，我们不能再仅依赖于集群特定的 IP 地址范围，因为外部网络无法识别这些地址范围。相反，我们需要一种方法将外部可路由的 IP 地址分配给我们的服务，可以通过显式地将 IP 地址与服务关联，或者使用 *ingress 控制器* 来监听外部流量并将其路由到服务。

#### 外部服务

我们之前创建的 `nginx` Service 是一个 `ClusterIP` Service，它是默认的 Service 类型。Kubernetes 支持多种 Service 类型，包括为需要公开的服务而设计的类型：

None 也称为 *无头* 服务，用于启用对选定 Pod 的跟踪，但没有 IP 地址或任何网络路由行为。

ClusterIP 默认的 Service 类型，提供对选定 Pod 的跟踪，一个集群内路由的集群 IP 地址，以及在集群 DNS 中的一个知名名称。

NodePort 扩展了 `ClusterIP`，并为集群中的所有节点提供了一个端口，该端口路由到该服务。

LoadBalancer 扩展了 `NodePort`，并使用底层云提供商来获取一个外部可访问的 IP 地址。

ExternalName 在集群 DNS 中为一个知名服务名称设置别名，指向某个外部 DNS 名称。用于使外部资源看起来像集群内的服务。

在这些服务类型中，`NodePort` 和 `LoadBalancer` 类型最适合将服务暴露到集群外部。`LoadBalancer` 类型似乎最直接，因为它只是为服务添加一个外部 IP 地址。然而，它需要与底层云环境集成，以便在创建服务时创建外部 IP 地址，将流量从该 IP 地址路由到集群的节点，并在集群外创建 DNS 记录，使外部用户能够在我们已经拥有的预注册域名下找到该服务，而不是仅在集群内有效的 `cluster.local` 域名。

因此，`LoadBalancer` 服务对于我们知道所使用的云环境，并且我们创建的服务会长期存在的情况最为有用。对于 HTTP 流量，我们可以通过将 `NodePort` 服务与入口控制器一起使用，获得 `LoadBalancer` 服务的大部分好处，并且还可以更好地支持动态部署新的应用程序和服务。

在继续讨论入口控制器之前，让我们将现有的 `nginx` 服务转换为 `NodePort` 服务，这样我们就可以查看效果。我们可以通过使用补丁文件来实现这一点：

*nginx-nodeport.yaml*

```
---
spec:
  type: NodePort
```

补丁文件允许我们仅更新我们关心的特定字段。在这种情况下，我们只更新服务的类型。为了使其生效，我们只需要在正确的位置指定一个更改的字段，这样 Kubernetes 就能知道该修改哪个字段。我们不需要更改服务的选择器或端口，只需要更改类型，因此补丁非常简单。

让我们使用这个补丁：

```
root@host01:~# kubectl patch svc nginx --patch-file /opt/nginx-nodeport.yaml 
service/nginx patched
```

对于这个命令，我们必须指定要补丁的资源和要使用的补丁文件。其结果与我们编辑服务的 YAML 资源文件并再次使用 `kubectl apply` 相同。

服务现在看起来有点不同：

```
root@host01:~# kubectl get service nginx
NAME    TYPE       CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE
nginx   NodePort   10.100.221.220   <none>        80:31326/TCP   2h
```

`NodePort` 服务提供了 `ClusterIP` 服务的所有行为，因此我们仍然拥有与我们的 `nginx` 服务关联的集群 IP。服务甚至保留了相同的集群 IP。唯一的变化是 `PORT` 字段现在显示服务端口 80 被附加到节点端口 31326。

每个集群节点上的 `kube-proxy` 服务正在监听这个端口（请确保使用适合你服务的正确节点端口）：

```
root@host01:~# ss -nlp | grep 31326
tcp   LISTEN 0  4096  .0.0.0:31326 ... users:(("kube-proxy",pid=3339,fd=15))
```

结果是，我们仍然可以在我们的 Pod 内部使用 `nginx` 服务名，但也可以使用来自主机的 NodePort：

```
root@host01:~# kubectl exec -ti pod -- wget -O - http://nginx
Connecting to nginx (10.100.221.220:80)
...
<title>Welcome to nginx!</title>
...
root@host01:~# wget -O - http://host01:31326
...
Connecting to host01 (host01)|127.0.2.1|:31326... connected.
...
<h1>Welcome to nginx!</h1>
...
```

因为`kube-proxy`监听所有网络接口，我们已经成功地将这个服务暴露给外部用户。

#### Ingress 服务

尽管我们已成功将 NGINX 服务暴露到集群外部，但仍然没有为外部用户提供良好的用户体验。要使用`NodePort`服务，外部用户需要知道至少一个集群节点的 IP 地址，并且需要知道每个服务监听的精确端口。如果该服务被删除并重新创建，则该端口可能会发生变化。我们可以通过告诉 Kubernetes 使用哪个端口来部分解决这个问题，但我们不想对任何任意服务这么做，因为多个服务可能会选择相同的端口。

我们真正需要的是一个单一的外部入口点，用于跟踪可用的多个服务，并使用规则将流量路由到它们。这样，我们就可以在集群内部完成所有的路由配置，以便服务可以动态地来来去去。同时，我们可以为我们的集群提供一个单一的、所有外部用户都能使用的入口点。

对于 HTTP 流量，Kubernetes 提供了正是这种能力，称之为*Ingress*。为了配置我们的集群将外部 HTTP 流量路由到服务，我们需要定义一组 Ingress 资源，指定路由规则，并部署接收和路由流量的入口控制器。当我们设置集群时，我们已经安装了入口控制器：

```
root@host01:~# kubectl -n ingress-nginx get deploy
NAME                       READY   UP-TO-DATE   AVAILABLE   AGE
ingress-nginx-controller   1/1     1            1           15d
root@host01:~# kubectl -n ingress-nginx get svc
NAME                      TYPE        ... PORT(S)               ...
ingress-nginx-controller  NodePort    ... 80:80/TCP,443:443/TCP ...
...
```

我们的入口控制器包括一个部署和一个服务。由于该服务类型为`NodePort`，我们知道`kube-proxy`正在集群所有节点的 80 和 443 端口上监听，准备将流量路由到相关的 Pod。

顾名思义，我们的入口控制器实际上是一个 NGINX web 服务器的实例；然而，在这种情况下，NGINX 仅作为 HTTP 反向代理，而不提供任何自己的网页内容。入口控制器监听集群中 Ingress 资源的变化，并根据定义的规则重新配置 NGINX，以连接到后端服务器。这些规则使用 HTTP 请求中的主机或路径信息来选择为该请求提供服务的服务。

让我们创建一个 Ingress 资源，将流量路由到我们在示例 9-1 中定义的`nginx`服务。以下是我们将创建的资源：

*nginx-ingress.yaml*

```
---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: web01
spec:
  rules:
    - host: web01
      http:
        paths:
          - path: /
 pathType: Prefix
            backend:
              service:
                name: nginx
                port:
                  number: 80
```

这个资源指示入口控制器查看 HTTP `Host`头部。如果它看到`web01`作为`Host`头部，它就会尝试与我们指定的`paths`中的路径进行匹配。在这种情况下，所有路径都会匹配`/`前缀路径，因此所有流量都会路由到`nginx`服务。

在将其应用到集群之前，让我们确认一下如果我们尝试使用一个入口控制器无法识别的主机名会发生什么。我们将使用与集群相关联的高可用性 IP 地址，因为集群的负载均衡器会将其转发到其中一个实例：

```
root@host01:~# curl -vH "Host:web01" http://192.168.61.10
...
> Host:web01
...
<head><title>404 Not Found</title></head>
...
```

`curl` 命令中的 `-H "Host:web01"` 标志告诉 `curl` 在 HTTP 请求中使用 `host01` 作为 `Host` 头部的值。考虑到我们示例集群中没有能够将 `web01` 转换为集群 IP 地址的 DNS 服务器，这是必要的。

如我们所见，作为 Ingress 控制器的 NGINX 服务器被配置为每当收到不匹配任何已配置的 Ingress 资源的请求时，都会回复一个 `404 Not Found` 错误信息。在这种情况下，因为我们还没有创建任何 Ingress 资源，所以任何请求都会得到这个响应。

让我们将 `web01` Ingress 资源应用到集群中：

```
root@host01:~# kubectl apply -f /opt/nginx-ingress.yaml 
ingress.networking.k8s.io/web01 created
```

现在既然已存在 Ingress 资源，如清单 9-3 所示，集群的高可用性 IP 和各个主机上的 HTTP 80 端口请求都会被路由到 `nginx` 服务：

```
root@host01:~# curl -vH "Host:web01" http://host01
...
> Host:web01
...
<title>Welcome to nginx!</title>
...
root@host01:~# curl -vH "Host:web01" http://192.168.61.10
...
> Host:web01
...
<title>Welcome to nginx!</title>
...
```

*清单 9-3：通过 Ingress 配置 NGINX*

两种情况的输出是相同的，显示流量正被路由到 `nginx` 服务。

在 `web01-ingress` 资源中，我们能够使用 `nginx` 服务的裸名称。服务名称的查找是基于 Ingress 资源所在的位置。由于我们在默认的命名空间中创建了 Ingress 资源，因此它会首先在该命名空间中查找服务。

将这一切结合起来，我们现在有了一个高可用性解决方案，将外部用户的流量路由到集群中的 HTTP 服务器。这将集群的高可用性 IP 地址 `192.168.61.10` 与暴露为 `NodePort` 服务的 Ingress 控制器结合在一起，该服务位于集群所有节点的 80 端口。Ingress 控制器可以通过创建新的 Ingress 资源动态配置以暴露其他服务。

#### 生产环境中的 Ingress

清单 9-3 中的 `curl` 命令看起来仍然有点奇怪，因为我们需要手动覆盖 HTTP `Host` 头。为了在生产集群中使用 Ingress 资源暴露服务，我们还需要执行一些额外的步骤。

首先，我们需要让集群拥有一个可外部路由的 IP 地址，并且这个地址需要有一个在 DNS 中注册的知名名称。做到这一点的最佳方法是使用通配符 DNS 方案，使得给定域名下的所有主机都路由到集群的外部 IP。例如，如果我们拥有 `cluster.example.com` 域名，我们可以创建一个 DNS 条目，使得 `*.cluster.example.com` 路由到集群的外部 IP 地址。

这种方法在跨多个网络的大型集群中仍然有效。我们只需要为 DNS 条目关联多个 IP 地址，可能还需要使用基于位置的 DNS 服务器，将客户端路由到最接近的服务。

接下来，我们需要为我们的 Ingress 控制器创建一个 SSL 证书，该证书包含我们的通配符 DNS 作为主题备用名称（SAN）。这将使得我们的 Ingress 控制器能够为外部用户提供安全的 HTTP 连接，无论他们使用的是哪个特定的服务主机名。

最后，当我们定义我们的 Service 时，需要为 `host` 字段指定完全限定的域名。对于上述示例，我们应该指定 `web01.cluster.example.com`，而不是仅仅使用 `web01`。

在完成这些额外步骤之后，任何外部用户都可以通过 HTTPS 连接到我们 Service 的完全限定主机名，例如 `https://web01.cluster.example.com`。这个主机名会解析到我们集群的外部 IP 地址，负载均衡器会将流量路由到集群的某个节点。此时，我们的 Ingress 控制器会监听标准端口 443，提供其通配符证书，该证书与客户端的期望匹配。安全连接建立后，Ingress 控制器会检查 HTTP `Host` 头，并将连接代理到正确的 Service，随后将 HTTP 响应返回给客户端。

这种方法的优点是，一旦我们完成设置，就可以随时部署新的 Ingress 资源来将 Service 暴露到外部，只要我们选择一个唯一的主机名，就不会与任何其他暴露的 Service 冲突。在初始设置完成后，所有配置都保存在集群内部，我们仍然为所有 Service 保持高度可用的配置。

### 最后思考

在 Kubernetes 集群中路由网络流量可能涉及相当复杂的操作，但最终结果是直接的：我们可以将应用程序组件部署到集群中，并实现自动扩展和故障转移，外部用户可以使用一个广为人知的名称访问我们的应用，而不需要知道应用程序是如何部署的，或者我们使用了多少个容器实例来满足需求。如果我们将应用程序构建得具有弹性，那么我们的应用程序容器可以在不影响用户的情况下升级到新版本或因故障重启。

当然，如果我们要构建具有弹性的应用程序组件，那么了解容器部署过程中可能出现的问题非常重要。在下一章中，我们将讨论一些在 Kubernetes 集群中部署容器时常见的问题以及如何调试这些问题。
