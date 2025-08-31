# 第十一章：GraphQL API 测试检查表

## 侦察

1.  使用 Nmap 进行端口扫描，以识别开放的 Web 应用程序端口。

1.  使用 Graphw00f 的检测模式扫描 Web 服务器的 GraphQL 端点。

1.  使用 Graphw00f 的指纹识别模式进行服务器指纹识别。

1.  在 MITRE 的 CVE 数据库中搜索服务器级别的漏洞。

1.  在 GraphQL 威胁矩阵中搜索服务器级别的安全功能。

1.  使用 EyeWitness 搜索 GraphQL IDE，如 GraphiQL Explorer 或 GraphQL Playground。

1.  发送一个自省查询并记录所有可用的查询、变异和订阅。

1.  使用 GraphQL Voyager 可视化自省查询响应。

## 拒绝服务

1.  审查 API 的 SDL 文件以查找双向关系。

1.  测试以下内容：

    1.  循环查询或变异

    1.  循环片段

    1.  字段重复

    1.  别名重载

    1.  指令重载

    1.  基于数组或别名的查询批处理

    1.  在 API 分页参数中如`filter`、`max`、`limit`和`total`中的对象限制覆盖

## 信息泄露

1.  在禁用自省时，使用字段填充提取 GraphQL 架构。

1.  通过发送格式错误的查询来识别查询响应中的调试错误。

1.  在 GraphQL 响应中识别查询追踪。

1.  测试通过 GET 方法提交的任何 PII。

## 认证和授权

1.  测试以下内容的访问权限：

    1.  没有认证头的 API

    1.  使用替代路径访问受限字段

    1.  使用 GET 和 POST 方法测试 API

1.  测试 JSON Web Token（JWT）中的签名验证。

1.  尝试暴力破解接受秘密（如令牌或密码）的变异或查询，使用以下方法：

    1.  基于别名的查询批处理

    1.  基于数组的查询批处理

    1.  CrackQL

    1.  Burp Suite

## 注入

1.  测试以下内容的注入：

    1.  查询参数

    1.  字段参数

    1.  查询指令参数

    1.  操作名称

1.  使用 SQLmap 自动测试 SQL 注入（SQLi）。

1.  使用 Commix 自动测试操作系统命令注入（OS Command Injection）。

## 请求伪造

1.  测试以下内容：

    1.  HTTP 头或正文中反-CSRF 令牌的存在

    1.  可能的反-CSRF 令牌绕过

    1.  基于 GET 的查询的可用性

    1.  支持基于 GET 的变异

1.  在 GET 上执行改变状态的变异。

1.  在 POST 上执行改变状态的变异。

## 劫持请求

1.  确认 GraphQL 服务器是否执行以下操作：

    1.  支持订阅

    1.  在 WebSocket 握手期间验证`Origin`头
