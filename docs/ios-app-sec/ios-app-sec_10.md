## 第七章：7

**iOS 网络**

几乎所有应用都会使用三种 iOS 网络 API 中的一个或多个。按照抽象层级的顺序，它们分别是 URL 加载系统、Foundation `NSStream` API 和 Core Foundation `CFStream` API。URL 加载系统用于通过 URL 获取和操作数据，比如网络资源或文件。`NSStream` 和 `CFStream` 类则是稍低级的方法，用于处理网络连接，但它们并不直接到达套接字层级。这些类用于非 HTTP 基础的通信，或当你需要更直接控制网络行为时。

在本章中，我将详细讨论 iOS 网络，从高层次的 API 开始。对于大多数用途，应用可以使用高层次的 API，但也有一些情况是这些 API 不能完全满足需求的。然而，使用低层次的 API 时，需要考虑更多的陷阱。

### 使用 iOS URL 加载系统

URL 加载系统能够处理应用程序需要执行的大多数网络任务。与 URL API 交互的主要方式是构造一个 `NSURLRequest` 对象，并利用它实例化一个 `NSURLConnection` 对象，以及一个接收连接响应的代理。当响应完全接收后，代理会收到一个 `connection:didReceiveResponse` 消息，参数是一个 `NSURLResponse` 对象。^(1)

但并非每个人都能正确使用 URL 加载系统的功能，因此在本节中，我将首先展示如何发现一个绕过传输层安全性的应用。接着，你将学到如何通过证书验证端点，避免开放重定向的危险，并实现证书固定，限制你的应用信任的证书数量。

#### *正确使用传输层安全性*

*传输层安全性 (TLS)*，现代的替代 SSL 的规范，对于几乎所有网络应用的安全至关重要。正确使用 TLS 时，它不仅能确保通过连接传输的数据机密性，还能验证远程端点，确保呈现的证书是由受信任的证书颁发机构签名的。默认情况下，iOS 会做正确的事情™，拒绝连接任何拥有不受信任或无效证书的端点。但在各种应用中，无论是移动端还是其他类型，开发者经常明确禁用 TLS/SSL 端点验证，从而让应用的流量容易被网络攻击者拦截。

在 iOS 中，TLS 可以通过多种方式禁用。过去，开发者通常会使用 `NSURLRequest` 的未记录的私有类方法 `setAllowsAnyHTTPSCertificate` 来轻松禁用验证。苹果公司很快开始拒绝使用此方法的应用，就像它对使用私有 API 的应用所做的那样。然而，仍然存在一些混淆方法，可能会让这个 API 在审核过程中悄悄通过，因此需要检查代码库，确保该方法没有被其他名字调用。

还有一种更具灾难性的绕过 TLS 验证的方法。这也很可能会导致你的应用被拒绝，但它说明了类别的重要性。我曾经有一个客户，他们授权了一个本应相当简单的第三方代码，并将其包含在产品中。尽管该项目的其他地方都正确处理了 TLS，但他们更新后的第三方代码没有验证任何 TLS 连接。显然，第三方供应商实现了 `NSURLRequest` 的一个类别，使用 `allowsAnyHTTPSCertificateForHost` 方法来避免验证。该类别仅包含指令 `return YES;`，导致所有 `NSURLRequest` 安静地忽略错误的证书。这个教训是什么？测试代码，别做假设！另外，你必须审计第三方代码，就像审计你自己代码库中的其他代码一样。错误可能不是你的错，但没人会关心这个。

**注意**

*幸运的是，在 iOS 9 中，意外禁用 TLS 的错误变得更加困难，因为默认情况下，iOS 不允许应用进行非 TLS 连接。相反，开发者需要在应用的* Info.plist *中为通过明文 HTTP 访问的 URL 放置一个特定的例外。然而，这并不能解决故意禁用 TLS 保护的情况。*

现在，实际上有一个官方的 API 可以绕过 TLS 验证。你可以使用 `NSURLConnection` 的委托并实现 `NSURLConnectionDelegate` 协议。^(2) 委托必须实现 `willSendRequestForAuthenticationChallenge` 方法，然后可以调用 `continueWithoutCredentialForAuthenticationChallenge` 方法。这是当前的最新方法；你也可能会看到使用 `connection:canAuthenticateAgainstProtectionSpace:` 或 `connection:didReceiveAuthenticationChallenge:` 的旧代码。示例 7-1 显示了如何在实际中看到这样的做法。

```
- (void)connection:(NSURLConnection *)connection
     willSendRequestForAuthenticationChallenge:(NSURLAuthenticationChallenge *)
     challenge {
    NSURLProtectionSpace *space = [challenge protectionSpace];
    if([[space authenticationMethod] isEqualToString:NS
     URLAuthenticationMethodServerTrust]) {
        NSURLCredential *cred = [NSURLCredential credentialForTrust:
     [space serverTrust]];
        [[challenge sender] useCredential:cred forAuthenticationChallenge:
     challenge];
    }
}
```

*示例 7-1：响应挑战时发送虚拟的* `NSURLCredential` *对象*

这段代码看起来相当无害，特别是因为它在各处都使用了 *protection*、*credential*、*authentication* 和 *trust* 等词汇。实际上，它所做的是绕过 TLS 端点的验证，使连接容易受到拦截。

当然，我并不是鼓励你在应用程序中实际*做*任何绕过 TLS 验证的事情。你不应该这样做，如果你这么做，你就是个坏人。这些示例只是展示了你在检查代码时可能会看到的模式。这些模式可能很难发现和理解，但如果你看到绕过 TLS 验证的代码，务必进行修改。

#### *使用 NSURLConnection 的基本认证*

HTTP 基本认证并不是一种特别强大的认证机制。它不支持会话管理或密码管理，因此用户不能在不使用单独应用程序的情况下注销或更改密码。但对于某些任务，例如对 API 的认证，这些问题并不那么重要，你仍然可能会在应用程序的代码库中遇到这种机制，或者被要求自己实现它。

你可以使用`NSURLSession`或`NSURLConnection`来实现 HTTP 基本认证，但无论是编写应用程序还是检查他人的代码，你都需要注意几个陷阱。

最简单的实现使用`NSURLConnection`的`willSendRequestForAuthenticationChallenge`委托方法：

```
- (void)connection:(NSURLConnection *)connection
     willSendRequestForAuthenticationChallenge:(NSURLAuthenticationChallenge *)
     challenge {
    NSString *user = @"user";
    NSString *pass = @"pass";

    if ([[challenge protectionSpace] receivesCredentialSecurely] == YES &&
        [[[challenge protectionSpace] host] isEqualToString:@"myhost.com"]) {

    NSURLCredential *credential = [NSURLCredential credentialWithUser:user password
     :pass persistence:NSURLCredentialPersistenceForSession];

    [[challenge sender] useCredential:credential
           forAuthenticationChallenge:challenge];
    }
}
```

委托对象首先会传递一个`NSURLAuthenticationChallenge`对象。接着，它创建一个包含用户名和密码的凭证，用户名和密码可以由用户提供，或从钥匙串中获取。最后，挑战的发送者会将凭证和挑战一起返回。

实现 HTTP 基本认证时需要注意两个潜在问题。首先，避免将用户名和密码存储在源代码或共享偏好设置中。你可以使用`NSURLCredentialStorage` API 自动将用户提供的凭证存储在钥匙串中，使用`sharedCredentialStorage`，如示例 7-2 所示。

```
➊ NSURLProtectionSpace *protectionSpace = [[NSURLProtectionSpace alloc] initWithHost:
        @"myhost.com" port:443 protocol:@"https" realm:nil authenticationMethod:nil];

➋ NSURLCredential *credential = [NSURLCredential credentialWithUser:user password:
        pass persistence:NSURLCredentialPersistencePermanent];

➌ [[NSURLCredentialStorage sharedCredentialStorage] setDefaultCredential:credential
        forProtectionSpace:protectionSpace];
```

*示例 7-2：设置保护空间的默认凭证*

这会创建一个保护空间➊，其中包括主机、端口、协议，以及可选的 HTTP 认证领域（如果使用 HTTP 基本认证）和认证方法（例如，使用 NTLM 或其他机制）。在➋，示例创建一个凭证，凭证包含最有可能从用户输入中接收到的用户名和密码。然后，在➌，它将凭证设置为该保护空间的默认凭证，并且凭证应该会自动存储在钥匙串中。未来，属于该代码的应用程序可以使用相同的 API 通过`defaultCredentialForProtectionSpace`方法读取凭证，如示例 7-3 所示。

```
credentialStorage = [[NSURLCredentialStorage sharedCredentialStorage]
     defaultCredentialForProtectionSpace:protectionSpace];
```

*示例 7-3：使用保护空间的默认凭证*

需要注意的是，存储在 `sharedCredentialStorage` 中的凭证会被标记为钥匙串属性 `kSecAttrAccessibleWhenUnlocked`。如果你需要更严格的保护，你需要自行管理钥匙串存储。我在第十三章中讲解了如何管理钥匙串。

此外，请确保在创建凭证时，注意如何指定 `persistence` 参数的值。如果你使用 `NSURLCredentialStorage` 存储在钥匙串中，可以在创建凭证时使用 `NSURLCredentialPersistencePermanent` 或 `NSURLCredentialPersistenceSynchronizable` 类型。如果你是用认证做一些更临时的操作，`NSURLCredentialPersistenceNone` 或 `NSURLCredentialPersistenceForSession` 类型更为合适。你可以在表 7-1 中找到每种持久性类型的详细含义。

**表 7-1：** 凭证持久性类型

| **持久性类型** | **含义** |
| --- | --- |
| `NSURLCredentialPersistenceNone` | 完全不存储凭证。仅在你需要对受保护资源进行一次请求时使用此项。 |
| `NSURLCredentialPersistenceForSession` | 将凭证存储在应用程序的生命周期内。 |
| `NSURLCredentialPersistencePermanent` | 将凭证存储在钥匙串中。 |
| `NSURLCredentialPersistenceForSession` | 将凭证存储在应用程序的生命周期内。若你只在应用运行时需要凭证，可以使用此项。 |
| `NSURLCredentialPersistencePermanent` | 将凭证存储在钥匙串中。当你希望在用户安装应用程序时，凭证能够持续存在时使用此项。 |
| `NSURLCredentialPersistenceSynchronizable` | 将凭证存储在钥匙串中，并允许其同步到其他设备和 iCloud。当你希望用户在设备之间传输凭证并且不担心将凭证发送到像 iCloud 这样的第三方时使用此项。 |

#### *使用 NSURLConnection 实现 TLS 双向认证*

执行客户端认证的最佳方法之一是使用客户端证书和私钥；然而，这在 iOS 上有些复杂。基本概念相对简单：实现 `willSendRequestForAuthenticationChallenge`（以前为 `didReceiveAuthenticationChallenge`）的代理，检查认证方法是否为 `NSURLAuthenticationMethodClientCertificate`，检索并加载证书和私钥，构建凭证，并使用凭证进行挑战。不幸的是，Cocoa 没有内置的 API 用于管理证书，因此你需要在 Core Foundation 中进行一些操作，像下面这个基本框架一样：

```
   - (void)connection:(NSURLConnection *) willSendRequestForAuthenticationChallenge:(
        NSURLAuthenticationChallenge *)challenge {
       if ([[[challenge protectionSpace] authenticationMethod] isEqualToString:NS
        URLAuthenticationMethodClientCertificate]) {

           SecIdentityRef identity;
           SecTrustRef trust;
➊         extractIdentityAndTrust(somep12Data, &identity, &trust);
           SecCertificateRef certificate;
➋         SecIdentityCopyCertificate(identity, &certificate);
➌         const void *certificates[] = { certificate };
➍         CFArrayRef arrayOfCerts = CFArrayCreate(kCFAllocatorDefault, certificates,
        1, NULL);

➎         NSURLCredential *cred = [NSURLCredential credentialWithIdentity:identity
        certificates:(__bridge NSArray*)arrayOfCerts
         persistence:NSURLCredentialPersistenceNone];
➏         [[challenge sender] useCredential:cred
                 forAuthenticationChallenge:challenge];
       }
   }
```

这个示例创建了一个 `SecIdentityRef` 和 `SecTrustRef`，以便可以将它们传递给 ➊ 处的 `extractIdentityAndTrust` 函数。这个函数会从一个 PKCS #12 数据块（文件扩展名为 *.p12*）中提取身份和信任信息。这些归档文件只是将一组加密对象集中存储在一个地方。

然后，代码将创建一个 `SecCertificateRef`，并从身份中提取证书 ➋。接着，它构建一个数组，包含在 ➌ 处的唯一证书，并创建一个 `CFArrayRef` 来存放该证书 ➍。最后，代码创建一个 `NSURLCredential`，将其身份和仅包含一个元素的证书数组传入 ➎，并将此凭据作为挑战的答案呈现 ➏。

你会注意到在 ➊ 处有一些不明确的描述。这是因为获取实际证书 p12 数据有几种不同的方法。你可以执行一次性引导，通过安全通道获取新生成的证书，或者在本地生成证书，或者从文件系统读取证书，或者从钥匙串中获取证书。获取 `somep12Data` 中使用的证书信息的一种方式是从文件系统中检索，方法如下：

```
NSData *myP12Certificate = [NSData dataWithContentsOfFile:path];
CFDataRef somep12Data = (__bridge CFDataRef)myP12Certificate;
```

存储证书的最佳位置当然是钥匙串；我会在 第十三章进一步讲解。

#### *修改重定向行为*

默认情况下，`NSURLConnection` 会在遇到 HTTP 重定向时跟随它。然而，当发生重定向时，它的行为是比较特殊的。当重定向被触发时，`NSURLConnection` 会将请求发送到新位置，并携带原始 `NSURLHttpRequest` 中的 HTTP 头信息。不幸的是，这也意味着你当前的原始域名的 Cookie 会被传递到新位置。因此，如果攻击者能够让你的应用访问一个接受任意 URL 作为重定向目标的页面，那么该攻击者就能窃取你的用户 Cookie，以及你应用可能存储在 HTTP 头中的任何其他敏感数据。这种漏洞被称为 *开放重定向*。

你可以通过在 iOS 4.3 及更早版本的 `NSURLConnectionDelegate` 上实现 `connect:willSendRequest: redirectResponse`^(3)，或者在 iOS 5.0 及更高版本的 `NSURLConnectionDataDelegate` 上实现此方法来修改这一行为。^(4)

```
   - (NSURLRequest *)connection:(NSURLConnection *)connection
                willSendRequest:(NSURLRequest *)request
               redirectResponse:(NSURLResponse *)redirectResponse
   {
       NSURLRequest *newRequest = request;
➊     if (![[[redirectResponse URL] host] isEqual:@"myhost.com"]) {
           return newRequest;
       }

       else {
➋         newRequest = nil;
           return newRequest;
       }
   }
```

在 ➊ 处，这段代码会检查你要重定向的域名是否与网站的名称相同。如果相同，它会继续正常执行。如果不同，它会将请求修改为 `nil` ➋。

#### *TLS 证书固定*

在过去几年中，关于证书颁发机构（CAs，负责担保我们日常遇到的 TLS 证书的实体）出现了一些令人担忧的发展。除了客户端应用程序信任的签名机构数量庞大外，CAs 还发生了几次显著的安全漏洞事件，包括签名密钥被泄露或颁发过于宽松的证书。这些漏洞使得任何拥有签名密钥的人都可以冒充任何 TLS 服务器，意味着他们可以成功且透明地读取或修改请求及其响应。

为了帮助缓解这些攻击，许多类型的客户端应用程序实现了 *证书固定*。这个术语可以指代多种不同的技术，但核心思想是通过编程限制应用程序信任的证书数量。您可以将信任限制为单一 CA（即您的公司用于签署服务器证书的 CA），限制为您用来创建自己证书的内部根 CA（即信任链的顶部），或者仅限于一个叶证书（信任链底部的一个特定证书）。

作为 SSL Conservatory 项目的一部分，我的同事 Alban Diquet 开发了一些方便的封装器，使您能够在应用程序中实现证书固定。（了解更多内容，请访问 *[`github.com/iSECPartners/ssl-conservatory`](https://github.com/iSECPartners/ssl-conservatory)*。）您可以编写自己的封装器，也可以使用现有的封装器；无论哪种方式，一个好的封装器可以使证书固定变得相当简单。例如，下面是如何通过 Alban 的封装器轻松实现证书固定：

```
➊ - (NSData*)loadCertificateFromFile:(NSString*)fileName {
       NSString *certPath = [[NSString alloc] initWithFormat:@"%@/%@", [[NSBundle
        mainBundle] bundlePath], fileName];
       NSData *certData = [[NSData alloc] initWithContentsOfFile:certPath];
       return certData;
   }

   - (void)pinThings {
   NSMutableDictionary *domainsToPin = [[NSMutableDictionary alloc] init];

➋ NSData *myCertData = [self loadCertificateFromFile:@"myCerts.der"];
   if (myCertData == nil) {
       NSLog(@"Failed to load the certificates");
       return;
       }

➌ [domainsToPin setObject:myCertData forKey:@"myhost.com"];

➍ if ([SSLCertificatePinning loadSSLPinsFromDERCertificates:domainsToPin] != YES) {
       NSLog(@"Failed to pin the certificates");
       return;
       }
   }
```

在 ➊ 处，这段代码简单地定义了一个方法，从 DER 格式的文件中加载证书到 `NSData` 对象，并在 ➋ 处调用此方法。如果加载成功，代码会将 `myCertData` 放入 `NSMutableDictionary` ➌ 中，并调用主 `SSLCertificatePinning` 类的 `loadSSLPinsFromDERCertificates` 方法 ➍。加载这些固定证书后，应用程序还需要实现一个 `NSURLConnection` 委托，如 列表 7-4 所示。

```
- (void)connection:(NSURLConnection *)connection
     willSendRequestForAuthenticationChallenge:(NSURLAuthenticationChallenge *)
     challenge {

    if([challenge.protectionSpace.authenticationMethod isEqualToString:NS
     URLAuthenticationMethodServerTrust]) {

        SecTrustRef serverTrust = [[challenge protectionSpace] serverTrust];
        NSString *domain = [[challenge protectionSpace] host];
        SecTrustResultType trustResult;

        SecTrustEvaluate(serverTrust, &trustResult);
        if (trustResult == kSecTrustResultUnspecified) {

            // Look for a pinned public key in the server's certificate chain
            if ([SSLCertificatePinning verifyPinnedCertificateForTrust:serverTrust
     andDomain:domain]) {

                // Found the certificate; continue connecting
                [challenge.sender useCredential:[NSURLCredential credentialForTrust
     :challenge.protectionSpace.serverTrust] forAuthenticationChallenge:challenge];
            }
            else {
                // Certificate not found; cancel the connection
                [[challenge sender] cancelAuthenticationChallenge: challenge];
            }
        }
        else {
            // Certificate chain validation failed; cancel the connection
            [[challenge sender] cancelAuthenticationChallenge: challenge];
        }
    }
}
```

*列表 7-4：一个* `NSURLConnection` *委托，用于处理证书固定逻辑*

这段代码简单地评估远程服务器提供的证书链，并将其与应用程序中包含的固定证书进行比较。如果找到固定证书，则连接继续；如果没有找到，则取消认证挑战过程。

按照所示实现您的代理后，所有`NSURLConnection`的使用应该检查确保它们绑定到您预定义列表中的域名和证书对。如果您感兴趣，可以在*[`github.com/iSECPartners/ssl-conservatory/tree/master/ios`](https://github.com/iSECPartners/ssl-conservatory/tree/master/ios)*找到其余的代码来实现您自己的证书钉扎。涉及的其他逻辑相当复杂，所以我无法在这里展示所有代码。

**注意**

*如果您很急，可以使用 SSL Conservatory 示例代码中的代理类进行子类化。*

到目前为止，我展示了围绕`NSURLConnection`的网络安全问题和解决方案。但从 iOS 7 开始，`NSURLSession`比传统的`NSURLConnection`类更为推荐。让我们更详细地看看这个 API。

### 使用 NSURLSession

`NSURLSession`类通常更受开发者青睐，因为它专注于使用网络*会话*，而不是`NSURLConnection`专注于单个请求。虽然`NSURLSession`在某种程度上扩大了`NSURLConnection`的范围，但它还通过允许在单个会话上设置配置，而不是在应用程序中全局设置配置，提供了更多的灵活性。一旦会话被实例化，它们将被分配个别任务来执行，使用`NSURLSessionDataTask`、`NSURLSessionUploadTask`和`NSURLSessionDownloadTask`类。

在本节中，您将探索一些使用`NSURLSession`的方法，一些潜在的安全陷阱，以及一些`NSURLConnection`未提供的安全机制。

#### *NSURLSession 配置*

`NSURLSessionConfiguration`类封装了传递给`NSURLSession`对象的选项，以便您可以为不同类型的请求提供独立的配置。例如，您可以对获取不同敏感级别数据的请求应用不同的缓存和 cookie 策略，而不是让这些策略在整个应用程序中全局应用。要使用`NSURLSession`的系统策略，您可以使用默认策略`[NSURLSessionConfigurationdefaultConfiguration]`，或者您可以简单地不指定配置策略，直接使用`[NSURLSessionsharedSession]`来实例化请求对象。

对于那些不应在本地存储留下任何痕迹的安全敏感请求，应使用配置方法`ephemeralSessionConfiguration`。另一种方法`backgroundSessionConfiguration`专门用于长时间运行的上传或下载任务。这种类型的会话将交给系统服务来管理完成，即使您的应用被终止或崩溃。

此外，您可以首次指定连接仅使用 TLS 版本 1.2，这有助于防御 BEAST^(5)和 CRIME^(6)等攻击，这些攻击可能允许网络攻击者读取或篡改您的 TLS 连接。

**注意**

*会话配置在实例化* `*NSURLSession*` *后是只读的；会话期间无法更改策略和配置，且无法更换为不同的配置。*

#### *执行 NSURLSession 任务*

让我们一起看看创建`NSURLSessionConfiguration`并为其分配简单任务的典型流程，如示例 7-5 所示。

```
➊ NSURLSessionConfiguration *configuration = [NSURLSessionConfiguration
        ephemeralSessionConfiguration];

➋ [configuration setTLSMinimumSupportedProtocol = kTLSProtocol12];

➌ NSURL *url = [NSURL URLWithString:@"https://www.mycorp.com"];

   NSURLRequest *request = [NSURLRequest requestWithURL:url];

➍ NSURLSession *session = [NSURLSession sessionWithConfiguration:configuration
                                                         delegate:self
                                                    delegateQueue:nil];

➎ NSURLSessionDataTask *task = [session dataTaskWithRequest:request
                                           completionHandler:
        ^(NSData *data, NSURLResponse *response, NSError *error) {
➏         // Your completion handler block
        }];

➐ [task resume];
```

*示例 7-5：创建一个要求 TLSv1.2 的临时* `NSURLConfiguration`

在➊处实例化了`NSURLSessionConfiguration`对象，指定连接应为临时的。这应该能防止缓存数据写入本地存储。然后，在➋处，配置要求使用 TLS 1.2 版本，因为开发者控制着端点并且知道该端点支持该版本。接下来，就像`NSURLConnection`一样，创建了一个`NSURL`对象和一个带有该 URL 的`NSURLRequest`对象 ➌。创建配置和请求后，应用程序可以实例化会话 ➍并为该会话分配任务 ➎。

`NSURLSessionDataTask`及其相关对象将一个完成处理器块作为参数 ➏。这个块异步处理服务器响应和你因任务收到的数据。或者（或额外），你可以指定一个符合`NSURLSessionTaskDelegate`协议的自定义代理。你可能希望同时使用`completionHandler`和代理的原因之一是，完成处理器处理请求结果，而代理则在会话级别而非任务级别管理认证和缓存决策（我将在下一节讨论这个问题）。

最后，在➐处，这段代码通过调用`resume`方法启动任务，因为所有任务在创建时都会被暂停。

#### *发现 NSURLSession TLS 绕过*

`NSURLSession` 也有一种方法可以避免 TLS 检查。应用程序可以使用`didReceiveChallenge`代理，并将接收到的挑战的`proposedCredential`作为凭证传回给会话， 如示例 7-6 所示。

```
   - (void)URLSession:(NSURLSession *)session didReceiveChallenge:(NS
        URLAuthenticationChallenge *)challenge completionHandler:(void (^)(NS
        URLSessionAuthChallengeDisposition disposition, NSURLCredential * credential))
        completionHandler {

➊      completionHandler(NSURLSessionAuthChallengeUseCredential,
          [challenge proposedCredential]);
   }
```

*示例 7-6：使用* `NSURLSession` *绕过服务器验证*

这是另一个可能很难发现的绕过方法。查看像➊处那样的代码，其中有一个`completionHandler`，后面跟着`proposedCredential`。

#### *使用 NSURLSession 的基本认证*

使用`NSURLSession`进行 HTTP 认证由会话处理，并传递给`didReceiveChallenge`代理，如示例 7-7 所示。

```
➊ - (void)URLSession:(NSURLSession *)session didReceiveChallenge:(NS
        URLAuthenticationChallenge *)challenge completionHandler:(void (^)(NS
        URLSessionAuthChallengeDisposition, NSURLCredential *))completionHandler {
       NSString *user = @"user";
       NSString *pass = @"pass";

       NSURLProtectionSpace *space = [challenge protectionSpace];
        if ([space receivesCredentialSecurely] == YES &&
            [[space host] isEqualToString:@"myhost.com"] &&
            [[space authenticationMethod] isEqualToString:NS
        URLAuthenticationMethodHTTPBasic]) {

➋    NSURLCredential *credential =
        [NSURLCredential credentialWithUser:user
                                   password:pass
                                persistence:NSURLCredentialPersistenceForSession];

➌    completionHandler(NSURLSessionAuthChallengeUseCredential, credential);
       }
   }
```

*示例 7-7：一个示例的* `didReceiveChallenge` *代理*

这种方法在 ➊ 处定义了一个代理和一个完成处理程序，在 ➋ 处创建了一个 `NSURLCredential`，并将该凭据传递给 ➌ 处的完成处理程序。请注意，无论是 `NSURLConnection` 还是 `NSURLSession` 方法，一些开发者会忘记确保他们与正确的主机通信或以安全的方式发送凭据。这将导致凭据发送到你应用加载的 *每个* URL，而不仅仅是你自己的；清单 7-8 展示了这个错误可能是什么样子的。

```
- (void)URLSession:(NSURLSession *)session didReceiveChallenge:(NS
     URLAuthenticationChallenge *)challenge completionHandler:(void (^)(NS
     URLSessionAuthChallengeDisposition, NSURLCredential *))completionHandler {

    NSURLCredential *credential =
      [NSURLCredential credentialWithUser:user
                                 password:pass
                              persistence:NSURLCredentialPersistenceForSession];

    completionHandler(NSURLSessionAuthChallengeUseCredential, credential);
}
```

*清单 7-8：错误的 HTTP 认证方式*

如果你想为某个专用端点使用持久化凭据，可以像使用 `NSURLConnection` 一样将凭据存储在 `sharedCredentialStorage` 中。在构建会话时，你可以提前提供这些凭据，而无需担心代理方法，正如在清单 7-9 中所示。

```
NSURLSessionConfiguration *config = [NSURLSessionConfiguration
     defaultSessionConfiguration];
[config setURLCredentialStorage:
    [NSURLCredentialStorage sharedCredentialStorage]];

NSURLSession *session = [NSURLSession sessionWithConfiguration:config
                                                      delegate:nil
                                                 delegateQueue:nil];
```

*清单 7-9：使用* `NSURLSessionConfiguration` *引用存储的凭据*

这只是创建一个 `NSURLSessionConfiguration`，并指定它应使用共享凭据存储。当你连接到一个在钥匙串中存储了凭据的资源时，这些凭据将被会话使用。

#### *管理存储的 URL 凭据*

你已经看过如何使用 `sharedCredentialStorage` 存储和读取凭据，但 `NSURLCredentialStorage` API 也允许你使用 `removeCredential:forProtectionSpace` 方法移除凭据。例如，当用户明确决定从应用中退出或删除帐户时，你可能想要这样做。清单 7-10 展示了一个典型的使用场景。

```
NSURLProtectionSpace *space = [[NSURLProtectionSpace alloc]
     initWithHost:@"myhost.com"
             port:443
         protocol:@"https"
            realm:nil authenticationMethod:nil];

NSURLCredential *credential = [credentialStorage
     defaultCredentialForProtectionSpace:space];

[[NSURLCredentialStorage sharedCredentialStorage] removeCredential:credential
                                                forProtectionSpace:space];
```

*清单 7-10：移除默认凭据*

这将从你的本地钥匙串中删除凭据。然而，如果凭据的持久化类型为 `NSURLCredentialPersistenceSynchronizable`，凭据可能已通过 iCloud 同步到其他设备。要从所有设备中删除凭据，请使用 `NSURLCredentialStorageRemoveSynchronizableCredentials` 选项，正如在清单 7-11 中所示。

```
NSDictionary *options = [NSDictionary dictionaryWithObjects forKeys:NS
     URLCredentialStorageRemoveSynchronizableCredentials, YES];

[[NSURLCredentialStorage sharedCredentialStorage] removeCredential:credential
                                                forProtectionSpace:space
                                                           options:options];
```

*清单 7-11：从本地钥匙串和 iCloud 中移除凭据*

到此为止，你应该已经理解了 `NSURLConnection` 和 `NSURLSession` API 及其基本用法。你可能还会遇到其他网络框架，它们有自己的行为，并需要稍微不同的安全配置。我现在将介绍其中的一些。

### 第三方网络 API 的风险

在 iOS 应用中，有一些流行的第三方网络 API，主要用于简化各种网络任务，如多部分上传和证书固定。最常用的一个是 AFNetworking，^(7) 其次是现已过时的 ASIHTTPRequest。^(8) 在本节中，我将向你介绍这两个。

#### *AFNetworking 的不当与正确使用*

AFNetworking 是一个流行的库，构建于`NSOperation`和`NSHTTPRequest`之上。它提供了多种便捷方法来与不同类型的 Web API 交互并执行常见的 HTTP 网络任务。

与其他网络框架一样，一个关键任务是确保 TLS 安全机制没有被禁用。在 AFNetworking 中，TLS 证书验证可以通过几种方式禁用。其一是通过`_AFNETWORKING_ALLOW_INVALID_SSL_CERTIFICATES`标志，通常在*Prefix.pch*文件中设置。另一种方式是设置`AFHTTPClient`的一个属性，如示例 7-12 所示。

```
NSURL *baseURL = [NSURL URLWithString:@"https://myhost.com"];
AFHTTPClient* client = [AFHTTPClient clientWithBaseURL:baseURL];
[client setAllowsInvalidSSLCertificate:YES];
```

*示例 7-12：通过* `setAllowsInvalidSSLCertificate`禁用 TLS 验证

你可能看到的最后一种禁用 TLS 验证的方式是通过使用`setAllowsInvalidSSLCertificate`更改`AFHTTPRequestOperationManager`的安全策略，如示例 7-13 所示。

```
AFHTTPRequestOperationManager *manager = [AFHTTPRequestOperationManager manager];
[manager [securityPolicy setAllowInvalidCertificates:YES]];
```

*示例 7-13：使用* `securityPolicy`禁用 TLS 验证

你还需要确保你正在检查的代码在生产版本中没有使用`AFHTTPRequestOperationLogger`类。这个日志记录器在后台使用`NSLog`将请求的 URL 写入 Apple 系统日志，这使得其他应用程序在某些 iOS 版本中可以看到它们。

AFNetworking 提供的一个特别有用的功能是能够轻松地执行证书固定（certificate pinning）。你只需在项目的*.pch*文件中设置`_AFNETWORKING_PIN_SSL_CERTIFICATES_` #define，并适当设置`AFHTTPClient`实例的固定模式（`defaultSSLPinningMode`）属性；可用的模式在表 7-2 中描述。然后将你希望固定的证书放入捆绑包根目录，作为*.cer*扩展名的文件。

**表 7-2：** AFNetworking SSL 固定模式

| **模式** | **含义** |
| --- | --- |
| `AFSSLPinningModeNone` | 不执行证书固定，即使固定已启用。如果需要，可以在调试模式中使用。 |
| `AFSSLPinningModePublicKey` | 固定到证书的公钥。 |
| `AFSSLPinningModeCertificate` | 固定到提供的确切证书（或证书）。如果证书被重新签发，则需要更新应用程序。 |

如 AFNetworking 随附的示例代码所示，你可以检查 URL 来确定它们是否应该被固定。只需评估协议和域名，查看这些域名是否属于你。示例 7-14 展示了一个例子。

```
if ([[url scheme] isEqualToString:@"https"] &&
    [[url host] isEqualToString:@"yourpinneddomain.com"]) {
        [self setDefaultSSLPinningMode:AFSSLPinningModePublicKey];
    }

    else {
        [self setDefaultSSLPinningMode:AFSSLPinningModeNone];
    }

    return self;
}
```

*示例 7-14：确定一个 URL 是否应该被固定*

`else`语句并不是绝对必要的，因为不进行固定是默认设置，但它确实提供了某种清晰度。

请记住，AFNetworking 会固定捆绑包中提供的所有证书，但它不会检查证书的常用名称与网络端点的主机名是否匹配。如果你的应用程序同时绑定到多个具有不同安全标准的网站，这通常会成为一个问题。换句话说，如果你的应用程序同时绑定到 *[`funnyimages.com`](https://funnyimages.com)* 和 *[`www.bank.com`](https://www.bank.com)*，那么持有 *funnyimages.com* 私钥的攻击者就能够拦截你应用程序与 *bank.com* 之间的通信。

现在你已经对如何使用和滥用 AFNetworking 库有了一个初步了解，让我们继续讨论 ASIHTTPRequest。

#### *不安全的 ASIHTTPRequest 使用*

ASIHTTPRequest 是一个已弃用的库，类似于 AFNetworking，但功能稍微不那么完整，并且基于 CFNetwork API。它不应在新项目中使用，但你可能会在现有的代码库中找到它，尤其是在迁移成本过高的情况下。当检查这些代码库时，标准的 SSL 验证绕过方法是 `setValidatesSecureCertificate:NO`。

你还需要检查项目中的 *ASIHTTPRequestConfig.h*，以确保没有启用过于冗长的日志记录（见 Listing 7-15）。

```
// If defined, will use the specified function for debug logging
// Otherwise use NSLog
#ifndef ASI_DEBUG_LOG
    #define ASI_DEBUG_LOG NSLog
#endif

// When set to 1, ASIHTTPRequests will print information about what a request is
      doing
#ifndef DEBUG_REQUEST_STATUS
    #define DEBUG_REQUEST_STATUS 0
#endif

// When set to 1, ASIFormDataRequests will print information about the request body
      to the console
#ifndef DEBUG_FORM_DATA_REQUEST
    #define DEBUG_FORM_DATA_REQUEST 0
#endif

// When set to 1, ASIHTTPRequests will print information about bandwidth throttling
      to the console
#ifndef DEBUG_THROTTLING
    #define DEBUG_THROTTLING 0
#endif

// When set to 1, ASIHTTPRequests will print information about persistent
      connections to the console
#ifndef DEBUG_PERSISTENT_CONNECTIONS
    #define DEBUG_PERSISTENT_CONNECTIONS 0
#endif

// When set to 1, ASIHTTPRequests will print information about HTTP authentication
      (Basic, Digest or NTLM) to the console
#ifndef DEBUG_HTTP_AUTHENTICATION
    #define DEBUG_HTTP_AUTHENTICATION 0
#endif
```

*Listing 7-15：* ASIHTTPRequestConfig.h 中的日志定义

如果你确实希望使用这些日志功能，可能需要将它们包装在 `#ifdef DEBUG` 条件语句中，如下所示：

```
#ifndef DEBUG_HTTP_AUTHENTICATION
    #ifdef DEBUG
        #define DEBUG_HTTP_AUTHENTICATION 1
    #else
        #define DEBUG_HTTP_AUTHENTICATION 0
    #endif
#endif
```

这个 *ASIHTTPRequestConfig.h* 文件将日志功能包装在条件语句中，以防止在生产版本中泄露这些信息。

### Multipeer Connectivity

iOS 7 引入了 Multipeer Connectivity^(9)，它允许附近的设备在最小网络配置下进行通信。Multipeer Connectivity 的通信可以通过 Wi-Fi（点对点或多点网络）或蓝牙个人区域网络（PANs）进行。Bonjour 是浏览和广告可用服务的默认机制。

开发者可以使用 Multipeer Connectivity 来执行点对点文件传输或在设备之间流式传输内容。与任何类型的对等通信一样，验证来自不信任对等方的传入数据至关重要；然而，也有传输安全机制确保数据不被窃听。

Multipeer Connectivity 会话是通过 `MCSession` 类的 `initWithPeer` 或 `initWithPeer:securityIdentity:encryptionPreference:` 类方法创建的。后者方法允许你要求加密，并可以包含证书链来验证设备。

当为`encryptionPreference`指定值时，可选择的选项有`MCEncryptionNone`、`MCEncryptionRequired`和`MCEncryptionOptional`。请注意，这些选项与`0`、`1`或`2`的值可以互换。因此，尽管`0`和`1`的值表现得像布尔值一样，但`2`的值在功能上等同于完全没有加密。

强烈建议无条件要求加密，因为`MCEncryptionOptional`容易受到降级攻击。（你可以在 Alban Diquet 的 Black Hat 演讲中找到关于反向工程 Multipeer Connectivity 协议的更多细节^(10))。 列表 7-16 展示了一个典型的调用，创建一个会话并要求加密。

```
MCPeerID *peerID = [[MCPeerID alloc] initWithDisplayName:@"my device"];

MCSession *session = [[MCSession alloc] initWithPeer:peerID
                                    securityIdentity:nil
                                encryptionPreference:MCEncryptionRequired];
```

*列表 7-16：创建一个* `MCSession`

当连接到远程设备时，会调用委托方法`session:didReceiveCertificate:fromPeer:certificateHandler:`，传入对等方的证书，并允许你指定一个处理方法，根据证书是否成功验证来采取特定行动。

**注意**

*如果你未能创建* `*didReceiveCertificate*` *委托方法或没有在该委托方法中实现* `*certificateHandler*` *，则远程端点不会进行验证，这使得连接容易被第三方拦截。*

在检查使用 Multipeer Connectivity API 的代码库时，确保所有`MCSession`实例化时都提供身份并要求传输加密。任何包含敏感信息的会话绝不能仅仅使用`initWithPeer`实例化。还要确保`didReceiveCertificate`的委托方法存在并正确实现，并确保当对等方证书验证失败时，`certificateHandler`能够正确处理。你*不*希望看到像这样的代码：

```
- (void) session:(MCSession *)session didReceiveCertificate:(NSArray *)certificate
     fromPeer:(MCPeerID *)peerID
     certificateHandler:(void (^)(BOOL accept))certificateHandler
{
    certificateHandler(YES);
}
```

这段代码盲目地将`YES`布尔值传递给处理器，这是你绝对不应该做的。

你可以自行决定如何实现验证。验证系统往往是定制化的，但你有几种基本选项。你可以让客户端自行生成证书，然后使用*首次信任（TOFU）*，这只会验证所呈现的证书是否与第一次配对时展示的证书相同。你也可以实现一个服务器，当查询时返回用户的公钥证书，从而集中管理身份。选择一个适合你的业务模型和威胁模型的解决方案。

### 使用 NSStream 进行低级网络编程

`NSStream` 适用于建立非 HTTP 网络连接，但它也可以通过相对较少的工作用于 HTTP 通信。由于某些无法理解的原因，在 OS X Cocoa 和 iOS Cocoa Touch 之间的过渡中，Apple 删除了允许 `NSStream` 建立与远程主机网络连接的方法 `getStreamsToHost`。所以如果你想自己进行流式传输，那太棒了。否则，在技术问答 QA1652 中，^(11) Apple 描述了一个类别，你可以用它定义一个大致等同于 `NSStream` 的 `getStreamsToHostNamed` 方法。

另一种选择是使用较低级别的 Core Foundation `CFStreamCreatePairWithSocketToHost` 函数，并将输入和输出的 `CFStream` 强制转换为 `NSStream`，如 列表 7-17 所示。

```
NSInputStream *inStream;
NSOutputStream *outStream;

CFReadStreamRef readStream;
CFWriteStreamRef writeStream;
CFStreamCreatePairWithSocketToHost(NULL, (CFStringRef)@"myhost.com", 80, &
     readStream, &writeStream);
inStream = (__bridge NSInputStream *)readStream;
outStream = (__bridge NSOutputStream *)writeStream;
```

*列表 7-17：将* `CFStreams` *转换为* `NSStreams`

`NSStream` 只允许用户对连接的特性进行少量控制，如 TCP 端口和 TLS 设置（参见 列表 7-18）。

```
   NSHost *myhost = [NSHost hostWithName:[@"www.conglomco.com"]];

   [NSStream getStreamsToHostNamed:myhost
                              port:443
                       inputStream:&MyInputStream
                      outputStream:&MyOutputStream];

➊ [MyInputStream setProperty:NSStreamSocketSecurityLevelTLSv1
                      forKey:NSStreamSocketSecurityLevelKey];
```

*列表 7-18：使用* `NSStream` *打开基本的 TLS 连接*

这是 `NSStream` 的典型用法：设置主机、端口和输入输出流。由于你对 TLS 设置没有太多控制，唯一可能出错的设置是 ➊，`NSStreamSocketSecurityLevel`。你应该将其设置为 `NSStreamSocketSecurityLevelTLSv1`，以确保你不会使用过时的、已损坏的 SSL/TLS 协议。

### 使用 CFStream 进行更低级别的网络编程

使用 `CFStream` 时，开发者在 TLS 会话协商中有不幸过多的控制权。^(12) 参见 表 7-3，了解你应该查找的多个 `CFStream` 属性。这些控制允许开发者覆盖或禁用验证对等方的规范名称（CN）、忽略过期日期、允许不受信任的根证书，并完全忽略验证证书链。

**表 7-3：** 恶心的 `CFStream` TLS 安全常量

| **常量** | **含义** | **默认值** |
| --- | --- | --- |
| `kCFStreamSSLLevel` | 用于加密连接的协议。 | negotiated*^a* |
| `kCFStreamSSLAllowsExpiredCertificates` | 接受过期的 TLS 证书。 | false |
| `kCFStreamSSLAllowsExpiredRoots` | 接受证书链中包含已过期根证书的证书。 | false |
| `kCFStreamSSLAllowsAnyRoot` | 是否可以使用根证书作为 TLS 端点的证书（换句话说，自签名或未签名证书）。 | false |
| `kCFStreamSSLValidatesCertificateChain` | 是否验证证书链。 | true |
| `kCFStreamSSLPeerName` | 覆盖与证书的 CN 比较的主机名。如果设置为 `kCFNull`，则不执行验证。 | hostname |
| `kCFStreamSSLIsServer` | 此流是否作为服务器使用。 | false |
| `kCFStreamSSLCertificates` | 如果`kCFStreamSSLIsServer`为真，将使用的证书数组。 | 无 |

*a*. 默认常量是`kCFStreamSocketSecurityLevelNegotiatedSSL`，它会与服务器协商使用最强的可用方法。

你可能根本不应该使用这些安全常量，但如果你必须使用 TLS `CFStream`，就按照正确的方法操作。这很简单！前提是你没有在应用内创建网络服务器（这是`CFStream`在 iOS 应用中相当罕见的用法），你需要遵循两个步骤：

1.  将`kCFStreamSSLLevel`设置为`kCFStreamSocketSecurityLevelTLSv1`。

1.  不要搞乱其他任何东西。

### 结束语

你已经了解了许多应用与外界通信的方式，以及这些方式可能会被错误实现的情况。现在让我们把注意力转向与其他应用的通信，以及通过 IPC 传输数据时可能遇到的一些陷阱。
