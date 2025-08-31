# 第十三章：健康探针

![image](img/common01.jpg)

拥有一个可靠的应用程序不仅仅是让应用组件保持运行。应用组件还需要能够及时响应请求，并从依赖项中获取数据并发出请求。这意味着，“健康”应用组件的定义对于每个组件都是不同的。

同时，Kubernetes 需要知道 Pod 及其容器的健康状态，以便只将流量路由到健康的容器，并替换掉失败的容器。因此，Kubernetes 允许为容器配置自定义健康检查，并将这些健康检查集成到工作负载资源的管理中，例如 Deployment。

在本章中，我们将学习如何为我们的应用程序定义健康探针。我们将研究基于网络的健康探针和容器内部的探针。我们还将了解 Kubernetes 如何运行这些健康探针，并且当容器变得不健康时如何响应。

### 关于探针

Kubernetes 支持三种不同类型的探针：

**Exec** 运行一个命令或脚本来检查容器的状态。

**TCP** 确定一个套接字是否打开。

**HTTP** 验证 HTTP GET 是否成功。

此外，我们可以将这三种探针中的任何一种用于三种不同的用途：

**Liveness** 检测并重启失败的容器。

**Startup** 在启动活性探针之前，给容器额外的时间。

**Readiness** 在容器尚未准备好时避免向其发送流量。

在这三种用途中，最重要的是活性探针，因为它在容器的主要生命周期内运行，并可能导致容器重启。我们将详细了解活性探针，并利用这些知识理解如何使用启动探针和就绪探针。

### 活性探针

*活性*探针会在容器启动后持续运行。活性探针作为容器定义的一部分创建，任何未通过活性探针检查的容器将会被自动重启。

#### Exec 探针

首先从一个简单的活性探针开始，该探针会在容器内运行一个命令。Kubernetes 期望命令在超时前完成，并返回零表示成功，或者返回非零代码表示存在问题。

**NOTE**

*本书的示例仓库在* [`github.com/book-of-kubernetes/examples`](https://github.com/book-of-kubernetes/examples)。 *关于如何设置，请参见第 xx 页中的“运行示例”。*

让我们通过一个 NGINX web 服务器容器来说明这一点。我们将使用这个 Deployment 定义：

*nginx-exec.yaml*

```
------
kind: Deployment
apiVersion: apps/v1
metadata:
  name: nginx
spec:
  replicas: 1
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
        livenessProbe:
          exec:
            command: ["/usr/bin/curl", "-fq", "http://localhost"]
          initialDelaySeconds: 10
          periodSeconds: 5
```

`livenessProbe`的`exec`部分告诉 Kubernetes 在容器内运行一个命令。在这种情况下，使用`curl`并加上`-q`标志，这样它不会打印页面内容，而只是返回一个零退出代码表示成功。另外，`-f`标志使得`curl`对于任何 HTTP 错误响应（即 300 以上的响应码）返回非零退出代码。

`curl`命令每 5 秒运行一次，基于`periodSeconds`；它在容器启动后 10 秒开始，基于`initialDelaySeconds`。

本章的自动化脚本会将*nginx-exec.yaml*文件添加到*/opt*目录。按照平常的方式创建此部署：

```
root@host01:~# kubectl apply -f /opt/nginx-exec.yaml 
deployment.apps/nginx created
```

结果 Pod 的状态看起来和没有存活探针的 Pod 没有什么不同：

```
root@host01:~# kubectl get pods
NAME                     READY   STATUS    RESTARTS   AGE
nginx-68dc5f984f-jq5xl   1/1     Running   0          25s
```

然而，除了常规的 NGINX 服务器进程外，`curl`每 5 秒在容器内运行一次，验证是否可以连接到服务器。通过`kubectl describe`命令得到的详细输出展示了这一配置：

```
root@host01:~# kubectl describe deployment nginx
Name:                   nginx
Namespace:              default
...
Pod Template:
  Labels:  app=nginx
  Containers:
   nginx:
...
 Liveness:     exec [/usr/bin/curl -q http://localhost] delay=10s 
    timeout=1s period=5s #success=1 #failure=3
...
```

因为定义了存活探针，所以 Pod 持续显示`Running`状态且没有重启，这表明探针检查成功。`#success`字段显示一个成功的运行就足以认为容器是存活的，而`#failure`值显示连续三次失败会导致 Pod 被重启。

我们使用`-q`来丢弃`curl`的日志，但即使没有该标志，成功的存活探针的任何日志都会被丢弃。如果我们想保存存活探针的实时日志信息，我们需要将其发送到文件或使用日志库将其发送到网络。

在继续介绍其他类型的探针之前，让我们先看看如果存活探针失败会发生什么。我们将修补`curl`命令，尝试从服务器检索一个不存在的路径，这会导致`curl`返回非零退出代码，从而使我们的探针失败。

在第九章中我们使用了补丁文件来编辑 Service 类型。这里我们再做一次补丁以应用更改：

*nginx-404.yaml*

```
---
spec:
  template:
    spec:
      containers:
     ➊ - name: nginx
          livenessProbe:
            exec:
              command: ["/usr/bin/curl", "-fq", "http://localhost/missing"]
```

尽管补丁文件允许我们仅更新我们关心的特定字段，但在这种情况下，补丁文件有几行，因为我们需要指定完整的层次结构，并且还必须指定我们要修改的容器名称➊，以便 Kubernetes 将这些内容合并到该容器的现有定义中。

要补丁部署，请使用`kubectl patch`命令：

```
root@host01:~# kubectl patch deploy nginx --patch-file /opt/nginx-404.yaml 
deployment.apps/nginx patched
```

由于我们在部署中修改了 Pod 的规格，Kubernetes 需要终止旧的 Pod 并创建一个新的 Pod：

```
root@host01:~# kubectl get pods
NAME                     READY   STATUS        RESTARTS   AGE
nginx-679f866f5b-7lzsb   1/1     Terminating   0          2m28s
nginx-6cb4b995cd-6jpd7   1/1     Running       0          3s
```

最初，新 Pod 显示`Running`状态。然而，如果我们大约 30 秒后再次检查，会发现 Pod 出现了问题：

```
root@host01:~# kubectl get pods
NAME                     READY   STATUS    RESTARTS   AGE
nginx-6cb4b995cd-6jpd7   1/1     Running   1          28s
```

我们没有改变存活探针的初始延迟和周期，所以第一次探针是在 10 秒后启动，之后每 5 秒运行一次。需要三次失败才能触发重启，因此看到在 25 秒后出现一次重启并不奇怪。

Pod 的事件日志指出了重启的原因：

```
root@host01:~# kubectl describe pod
Name:         nginx-6cb4b995cd-6jpd7
...
Containers:
  nginx:
...
    Last State:     Terminated
...
Events:
  Type     Reason     Age                From     Message
  ----     ------     ----               ----     -------
...
  Warning  Unhealthy  20s (x9 over 80s)  kubelet  Liveness probe failed: ...
curl: (22) The requested URL returned error: 404 Not Found
...
```

事件日志提供了有用的 `curl` 输出，告诉我们存活探针失败的原因。Kubernetes 将继续每 25 秒重启容器，因为每个新容器启动后都会失败三个连续的存活探针。

#### HTTP 探针

在容器内运行命令以检查健康状况的能力使我们能够执行自定义探针。然而，对于像这样的 web 服务器，我们可以利用 Kubernetes 中的 HTTP 探针功能，避免在容器镜像内部使用 `curl`，同时验证从 Pod 外部的连接性。

让我们用一个新的配置替换我们的 NGINX Deployment，使用 HTTP 探针：

*nginx-http.yaml*

```
---
kind: Deployment
apiVersion: apps/v1
metadata:
  name: nginx
spec:
  replicas: 1
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
        livenessProbe:
          httpGet:
            path: /
            port: 80
```

在此配置中，我们告诉 Kubernetes 连接到 Pod 的 80 端口并在根路径 */* 上执行 HTTP GET。由于我们的 NGINX 服务器监听 80 端口并将在根路径上提供欢迎文件，我们可以期待它正常工作。

我们指定了整个 Deployment，而不是使用补丁，因此我们将使用 `kubectl apply` 来更新 Deployment：

```
root@host01:~# kubectl apply -f /opt/nginx-http.yaml 
deployment.apps/nginx configured
```

我们也可以使用补丁来进行这个更改，但这次会更复杂，因为补丁文件会被合并到现有配置中。因此，我们需要两个命令：一个删除现有的存活探针，另一个添加新的 HTTP 存活探针。最好是完全替换资源。

**注意**

*kubectl patch 命令是一个有价值的调试命令，但生产环境中的应用程序应该将 YAML 资源文件放在版本控制下，以便进行变更跟踪和同行审查，并且每次都应该应用整个文件，以确保集群反映当前仓库的内容。*

现在我们已经应用了新的 Deployment 配置，Kubernetes 会创建一个新的 Pod：

```
root@host01:~# kubectl get pods
NAME                    READY   STATUS    RESTARTS   AGE
nginx-d75d4d675-wvhxl   1/1     Running   0          2m38s
```

对于 HTTP 探针，`kubelet` 负责按适当的时间表运行 HTTP GET 请求并确认结果。默认情况下，任何 HTTP 返回码在 200 或 300 系列内都被视为成功响应。

NGINX 服务器会记录所有请求，因此我们可以使用容器日志来查看正在进行的探测：

```
root@host01:~# kubectl logs nginx-d75d4d675-wvhxl
...
... 22:23:31 ... "GET / HTTP/1.1" 200 615 "-" "kube-probe/1.21" "-"
... 22:23:41 ... "GET / HTTP/1.1" 200 615 "-" "kube-probe/1.21" "-"
... 22:23:51 ... "GET / HTTP/1.1" 200 615 "-" "kube-probe/1.21" "-"
```

这次我们没有指定 `periodSeconds`，所以 `kubelet` 以默认的每 10 秒一次的频率进行探测。

在继续之前，让我们先清理一下 NGINX Deployment：

```
root@host01:~# kubectl delete deployment nginx
deployment.apps "nginx" deleted
```

我们已经看过了三种探针中的两种，接下来我们来看看 TCP 探针。

#### TCP 探针

类似 PostgreSQL 这样的数据库服务器监听网络连接，但它不使用 HTTP 进行通信。我们仍然可以使用 TCP 探针为这些类型的容器创建探测。它不能提供 HTTP 或 exec 探针的配置灵活性，但它可以验证 Pod 中的容器是否在指定端口上监听连接。

这里是一个带有 TCP 探针的 PostgreSQL Deployment：

*postgres-tcp.yaml*

```
---
kind: Deployment
apiVersion: apps/v1
metadata:
  name: postgres
spec:
  replicas: 1
  selector:
    matchLabels:
      app: postgres
  template:
    metadata:
      labels:
        app: postgres
    spec:
      containers:
      - name: postgres
        image: postgres
        env:
        - name: POSTGRES_PASSWORD
          value: "supersecret"
        livenessProbe:
 tcpSocket:
            port: 5432
```

我们在 第十章 中看到需要 `POSTGRES_PASSWORD` 环境变量。这个例子中唯一更改的配置是 `livenessProbe`。我们指定了一个 5432 的 TCP 套接字，因为这是 PostgreSQL 的标准端口。

和往常一样，我们可以创建这个部署，并在一段时间后观察它是否正在运行：

```
root@host01:~# kubectl apply -f /opt/postgres-tcp.yaml 
deployment.apps/postgres created
...
root@host01:~# kubectl get pods
NAME                       READY   STATUS    RESTARTS   AGE
postgres-5566ff748-jqp5d   1/1     Running   0          29s
```

同样，执行探针的是 `kubelet`。它仅通过与端口建立 TCP 连接并断开连接来执行此操作。当发生这种情况时，PostgreSQL 不会输出任何日志，因此我们知道探针是否有效的唯一方式是检查容器是否继续运行且没有重启：

```
root@host01:~# kubectl get pods
NAME                       READY   STATUS    RESTARTS   AGE
postgres-5566ff748-jqp5d   1/1     Running   0          2m7s
```

在继续之前，让我们清理一下部署：

```
root@host01:~# kubectl delete deploy postgres
deployment.apps "postgres" deleted
```

我们现在已经查看了所有三种类型的探针。虽然我们使用这三种类型来创建存活探针，但这三种类型同样适用于启动探针和就绪探针。唯一的区别是当探针失败时，我们集群的行为会有所不同。

### 启动探针

不健康的容器可能会为应用程序带来各种困难，包括响应迟缓、请求响应错误或数据异常，因此我们希望 Kubernetes 在容器变为不健康时能迅速作出反应。然而，当容器首次启动时，可能需要一些时间才能完全初始化。在此期间，它可能无法响应存活探针。

由于这种延迟，我们需要在容器失败探针之前设置一个较长的超时时间，以便容器有足够的时间进行初始化。然而，同时我们又需要一个较短的超时时间，以便快速检测到失败的容器并重启它。解决方法是配置一个单独的 *启动探针*。Kubernetes 将使用启动探针配置，直到探针成功；然后它会切换到存活探针。

例如，我们可以如下配置我们的 NGINX 服务器部署：

```
...
spec:
...
  template:
...
    spec:
      containers:
      - name: nginx
        image: nginx
        livenessProbe:
          httpGet:
            path: /
            port: 80
        startupProbe:
          httpGet:
            path: /
            port: 80
          periodSeconds: 
          initialDelaySeconds: 30
          periodSeconds: 10
          failureThreshold: 60
```

根据这个配置，Kubernetes 将在启动后 30 秒开始检查容器。它将每 10 秒检查一次，直到探针成功或尝试达到 60 次失败。其效果是容器有 10 分钟的时间完成初始化并成功响应探针。如果容器在此时间内未能通过探针检查，它将被重启。

一旦容器成功通过一次探针，Kubernetes 就会切换到 `livenessProbe` 配置。因为我们没有重写任何定时参数，所以这将每 10 秒进行一次探针检查，连续三次探针失败将导致重启。我们最初为容器提供 10 分钟的存活时间，但之后将在 30 秒内没有响应时重启容器。

`startupProbe`被完全独立定义，这意味着可以为启动创建与活跃检查不同的检查。当然，重要的是要明智选择，以确保容器不会在活跃探针通过之前就通过其启动探针，因为那样会导致不适当的重启。

### 就绪探针

第三个探针的目的是检查 Pod 的*就绪性*。*就绪性*这个术语可能与启动探针显得有些重复。然而，尽管完成初始化是软件就绪性的一个重要部分，一个应用组件可能由于多种原因无法准备好执行工作，尤其是在一个高可用的微服务架构中，组件可能随时进出。

就绪探针应当用于任何容器无法执行工作因为超出其控制的故障的情况，而不是用于初始化。这个问题可能是暂时的，因为其他地方的重试逻辑可能会修复这个故障。例如，一个依赖外部数据库的 API 如果数据库无法访问，可能会失败其就绪探针，但该数据库可能随时恢复服务。

这也与启动和活跃探针形成了有价值的对比。如前所述，Kubernetes 将在容器未通过配置的启动或活跃探针次数时重启容器。但如果问题是外部依赖失败或缺失，重启容器毫无意义，因为重启容器无法解决外部的问题。

同时，如果一个容器缺少所需的外部依赖项，它就无法执行工作，因此我们不希望向它发送任何工作。在这种情况下，最好的做法是保持容器运行，并给予它重新建立所需连接的机会，但避免向它发送任何请求。与此同时，我们可以希望集群中其他地方有一个相同部署的 Pod 正常工作，使我们的应用整体对局部故障具有弹性。

这正是 Kubernetes 中就绪探针的工作原理。如我们在第九章中所看到的，Kubernetes 服务持续监视与其选择器匹配的 Pod，并为其集群 IP 配置负载均衡，将流量路由到这些 Pod。如果 Pod 报告自己未就绪，服务将停止将流量路由到它，但`kubelet`不会触发任何其他操作，如容器重启。

让我们来举一个例子。我们希望对 Pod 的就绪性进行单独控制，因此我们将使用一个稍微做作的例子，而不是一个真实的外部依赖来决定就绪性。我们将部署一组 NGINX Pod，并且这次会有一个相应的服务：

*nginx-ready.yaml*

```
---
kind: Deployment
apiVersion: apps/v1
metadata:
  name: nginx
spec:
  replicas: 3
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
        livenessProbe:
          httpGet:
            path: /
            port: 80
        readinessProbe:
          httpGet:
            path: /ready
            port: 80
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

此部署将其`livenessProbe`保留为 NGINX 正常工作的指标，并添加了一个`readinessProbe`。服务定义与第九章中看到的完全相同，并将流量路由到我们的 NGINX Pod。

此文件已写入*/opt*，因此我们可以将其应用到集群中：

```
root@host01:~# kubectl apply -f /opt/nginx-ready.yaml 
deployment.apps/nginx created
service/nginx created
```

这些 Pod 运行后会保持运行状态，因为存活探针成功：

```
root@host01:~# kubectl get pods
NAME                     READY   STATUS    RESTARTS   AGE
nginx-67fb6485f5-2k2nz   0/1     Running   0          38s
nginx-67fb6485f5-vph44   0/1     Running   0          38s
nginx-67fb6485f5-xzmj5   0/1     Running   0          38s
```

此外，我们创建的服务已分配了一个集群 IP：

```
root@host01:~# kubectl get services
NAME         TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)   AGE
...
nginx        ClusterIP   10.101.98.80   <none>        80/TCP    3m1s
```

然而，我们无法使用该 IP 地址访问任何 Pod：

```
root@host01:~# curl http://10.101.98.80
curl: (7) Failed to connect to 10.101.98.80 port 80: Connection refused
```

这是因为当前 NGINX 在*/ready*路径上没有内容可提供，因此返回`404`，就绪探针失败。对 Pod 的详细检查显示它尚未准备就绪：

```
root@host01:~# kubectl describe pod
Name:         nginx-67fb6485f5-2k2nz
...
Containers:
  nginx:
...
    Ready:          False
...
```

因此，服务没有任何端点可用于路由流量：

```
root@host01:~# kubectl describe service nginx
Name:              nginx
...
Endpoints:         
...
```

因为服务没有端点，已配置`iptables`拒绝所有流量：

```
root@host01:~# iptables-save | grep default/nginx
-A KUBE-SERVICES -d 10.101.98.80/32 -p tcp -m comment --comment "default/nginx has no endpoints"  
  -m tcp --dport 80 -j REJECT --reject-with icmp-port-unreachable
```

要解决此问题，我们需要至少一个 Pod 准备就绪，以确保 NGINX 有内容可以提供给*/ready*路径。我们将使用容器的主机名来跟踪哪个 Pod 正在处理我们的请求。

要使其中一个 Pod 准备就绪，让我们首先再次获取 Pod 列表，只是为了方便获取 Pod 名称：

```
root@host01:~# kubectl get pods
NAME                     READY   STATUS    RESTARTS   AGE
nginx-67fb6485f5-2k2nz   0/1     Running   0          10m
nginx-67fb6485f5-vph44   0/1     Running   0          10m
nginx-67fb6485f5-xzmj5   0/1     Running   0          10m
```

现在，我们将选择一个并使其报告为准备就绪：

```
root@host01:~# kubectl exec -ti nginx-67fb6485f5-2k2nz -- \
  cp -v /etc/hostname /usr/share/nginx/html/ready
'/etc/hostname' -> '/usr/share/nginx/html/ready'
```

我们的服务将开始显示一个有效的端点：

```
root@host01:~# kubectl describe svc nginx
Name:              nginx
...
Endpoints:         172.31.239.199:80
...
```

更好的是，我们现在可以通过集群 IP 访问 NGINX 实例，内容与主机名对应：

```
root@host01:~# curl http://10.101.98.80/ready
nginx-67fb6485f5-2k2nz
```

注意 URL 末尾的`/ready`，因此响应是主机名。如果多次运行此命令，我们会看到每次主机名都相同。这是因为通过存活探针的唯一 Pod 正在处理所有服务流量。

让我们也使其他两个 Pod 变为准备就绪状态：

```
root@host01:~# kubectl exec -ti nginx-67fb6485f5-vph44 -- \
  cp -v /etc/hostname /usr/share/nginx/html/ready
'/etc/hostname' -> '/usr/share/nginx/html/ready'
root@host01:~# kubectl exec -ti nginx-67fb6485f5-xzmj5 -- \
  cp -v /etc/hostname /usr/share/nginx/html/ready
'/etc/hostname' -> '/usr/share/nginx/html/ready'
```

我们的服务现在展示所有三个端点：

```
root@host01:~# kubectl describe service nginx
Name:              nginx
...
Endpoints:         172.31.239.199:80,172.31.239.200:80,172.31.89.210:80
...
```

多次运行`curl`命令显示流量现在分布在多个 Pod 之间：

```
root@host01:~# for i in $(seq 1 5); do curl http://10.101.98.80/ready; done
nginx-67fb6485f5-xzmj5
nginx-67fb6485f5-2k2nz
nginx-67fb6485f5-xzmj5
nginx-67fb6485f5-vph44
nginx-67fb6485f5-vph44
```

嵌入命令`$(seq 1 5)`返回数字一至五，导致`for`循环运行`curl`五次。如果多次运行相同的`for`循环，你将看到主机名的不同分布。如第九章所述，负载均衡基于随机均匀分布，每个端点被选中作为新连接的概率相等。

一个良好的实践是为每个应用程序提供一个 HTTP 准备就绪端点，检查应用程序及其依赖项的当前状态，并在组件健康时返回 HTTP 成功代码（如`200`），否则返回 HTTP 错误代码（如`500`）。某些应用框架（如 Spring Boot）提供自动公开存活和就绪端点的应用程序状态管理。

### 总结思路

Kubernetes 提供了检查我们的容器并确保它们按预期工作的能力，而不仅仅是进程正在运行。这些探针可以包括在容器内运行任意命令，验证端口是否开放以进行 TCP 连接，或者容器是否正确响应 HTTP 请求。为了构建弹性应用程序，我们应为每个应用程序组件定义一个存活探针和一个就绪探针。存活探针用于重新启动不健康的容器；就绪探针确定 Pod 是否能处理服务流量。此外，如果组件需要额外的初始化时间，我们还应定义一个启动探针，以确保在初始化完成后能够给予其所需的初始化时间，并在初始化完成后迅速响应失败。

当然，为了使我们的容器按预期运行，集群中的其他容器也必须表现良好，不能使用过多的集群资源。在下一章中，我们将看看如何限制我们的容器在使用 CPU、内存、磁盘空间和网络带宽方面，以及如何控制用户可用的总资源量。指定限制和配额的能力对于确保我们的集群能够支持多个应用程序并保持可靠的性能至关重要。
