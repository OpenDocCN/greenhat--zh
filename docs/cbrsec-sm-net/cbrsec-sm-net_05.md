# 第五章：# 创建虚拟私人网络

![章节开头图标](img/nsp-enoka501485-ct.jpg)

*虚拟私人网络（VPN）*是一种通过公共互联网提供通信隐私和安全性的手段。如果你不希望恶意第三方在你的谷歌搜索流量从本地笔记本电脑传输到谷歌服务器的过程中进行拦截，你应该使用 VPN 来加密两端之间的流量。如果你经常传输敏感文件或数据，如个人身份信息或银行信息，最好使用加密来保护这些信息。

VPN 的另一个主要功能是将私人网络（例如家庭和办公室中的网络）从一个地理位置扩展到另一个地理位置。VPN 在互联网上创建一个隧道，将一个网络与第二个网络连接。这意味着，如果一个通常位于澳大利亚的用户正在欧洲旅行，他们可以像在澳大利亚一样从欧洲连接到他们的家庭网络。相反，如果一个位于澳大利亚的用户希望*看起来*像是位于欧洲，他们可以将 VPN 端点设置在欧洲，通常通过某个第三方服务。

本章概述了一种创建私人 VPN 的方法，其中的*出口节点*（即 VPN 隧道结束的地方）位于你的本地网络之外，地理位置上位于世界某个地方，从而使你的实际物理位置保持隐私。我们将讨论如何使用 OpenVPN 或 Wireguard 实现这一目标。

## 第三方 VPN 和远程访问服务的缺点

尽管你可以订阅像 NordVPN 或 ExpressVPN 这样的 VPN 服务，运营自己的 VPN 更有利，因为你可以控制其中的一切，包括连接和流量记录的级别，以及服务的成本。此外，虽然第三方服务提供了一些好处，例如使用位于不同位置的多个出口节点的可能性，但它们通常不提供连接到你自己网络的远程访问功能。使用第三方 VPN 服务的最后一个挑战是，它们通常限制你可以同时连接的设备数量。私人管理的 VPN 没有这样的限制。

最近，许多应用程序涌现出来，旨在允许从更广泛的互联网远程访问端点。这些包括 Teamviewer 和 AnyDesk 等软件和供应商。尽管这些解决方案方便且入门门槛较低，但它们通过为你的计算机打开远程访问到互联网，增加了你的私人网络的攻击面，而这种行为应尽量避免，甚至从不做过。此外，已经发生了几起这些解决方案被攻破的著名事件，表明它们易受攻击。VPN 提供了一个更加安全的解决方案。

## OpenVPN

*OpenVPN*是最常见的 VPN 解决方案之一。由于其历史悠久且普及，您可以确信它在安全性方面优于较新的解决方案，因为后者在漏洞和安全缺陷的测试上不够严格。OpenVPN 已内置于多种网络硬件中，这一点非常有利，因为在很多情况下，您的路由器可以作为您网络内部的 VPN 端点（即 VPN 服务器）。这意味着您的路由器也可以充当 VPN 客户端，连接到云中的 VPN 服务器，然后您内部网络上连接到路由器的所有设备都可以通过 VPN 隧道发送和接收流量。以这种方式加密您的互联网流量，比不使用 VPN 直接上网提供了更大的隐私保护。然而，理想情况下，您希望对 VPN 出口节点有更多控制，因为大多数路由器使用的是简化版的 Linux 或专有操作系统，因此您将学习如何使用 Ubuntu 创建一个 VPN 服务器，以获得更大的灵活性。

## EasyRSA

EasyRSA 是一个命令行工具，用于创建和管理证书授权机构。为了加密和保护流量，OpenVPN 需要一个*证书授权机构（CA）*来颁发证书。*数字证书*用于建立不同方之间的信任，通常是网络和计算机之间的信任。*公钥基础设施（PKI）*负责公钥证书的分发、认证和撤销，这些证书用于验证数字证书的所有权。这些证书包含一个实体用作公私钥对一部分的公钥，用于加密数据，只有拥有匹配私钥的公钥所有者才能解密。这种方法保护了今天互联网上的大多数通信。

您创建的 CA 将生成、签名、验证并撤销（如有必要）所有用于加密和保护 VPN 服务器与 VPN 客户端之间通信的证书。从技术上讲，您可以将 OpenVPN 和 CA 安装在同一台服务器上，但这样做比将它们安装在不同的服务器上更不安全。任何能够访问该服务器的对手都能获取服务器使用的证书和私钥，并且有能力生成新的证书。因此，您需要两台 Ubuntu 服务器：一台作为 OpenVPN 服务器，另一台作为证书服务器。您将使用证书服务器来签署在 OpenVPN 服务器上生成的请求，这些请求既包括 VPN 服务器的请求，也包括任何连接到 VPN 的客户端设备的请求，无论它们是笔记本电脑、工作站、移动设备还是其他类型的设备。

## Wireguard

*Wireguard*，作为 OpenVPN 的一个相对较新的替代品，比起 OpenVPN 来说，它非常简单且速度极快。它较新的缺点是，尽管 Wireguard 是开源的，但由于开发时间较短，尚未经过充分的漏洞和错误测试。然而，它已经在安全社区中积累了相当大的用户群，并且因其可靠性和安全性而享有良好的声誉。

注意：如果你计划远程连接到你的私有网络，请记住，你需要在家里或办公室的互联网连接上拥有一个静态 IP 地址，并在边界路由器上进行一些端口转发。大多数互联网服务提供商可以按需提供静态 IP 地址，通常需要支付少量费用。

#19：使用 OpenVPN 创建 VPN

在这个第一个项目中，你将从创建一个 OpenVPN 服务器和一个证书授权机构开始，以通过 VPN 保护通信。接下来，你将生成相关证书，创建 OpenVPN 配置文件，配置主机防火墙，并启动 VPN。最后，你将配置每个将使用此 VPN 进行数据传输的 VPN 客户端，并连接和测试 VPN 连接。

在云端启动 OpenVPN 服务器并连接客户端的整个过程应该不会超过几个小时。为每个后续客户端添加连接大约需要每个端点 30 分钟。作为创建 VPN 的一部分，你需要启用并配置服务器上的防火墙。Ubuntu 内置的防火墙，*简单防火墙（UFW）*，旨在简化防火墙配置的复杂性。它比像 iptables（在第三章中讲解的）这样的解决方案要简单得多。在这个项目中，我们将介绍 UFW 及其用法，作为一种替代的主机防火墙解决方案。或者，你也可以应用在第三章中学到的知识，并在 iptables 部署中实现与 UFW 相同的规则。即使你已经部署了像 pfSense 这样的外围防火墙，仍然要确保启用 Ubuntu 提供的主机防火墙或 iptables，按照第三章中的内容，提供额外的主机级保护。实施主机防火墙还可以更精细地配置服务器的网络连接。

一旦启用防火墙，你需要调整 Ubuntu 安装的设置，以便 OpenVPN 流量能够穿越该防火墙。（我将在项目的后面部分讲解如何操作。）

为了保护来源于你网络内部的互联网流量，你将需要在其他地方配置一个 VPN 出口节点，以及一个证书服务器，因此请参照项目 3 和第一章，在云端创建两台基础的 Ubuntu 服务器，选择你偏好的云服务提供商。

一旦你的 Ubuntu 服务器启动并运行，作为标准的非 root 用户，通过 SSH 登录到你打算用作 OpenVPN 服务器（而不是证书颁发机构）的服务器：

```
$ `ssh` `user``@``your_server_IP`

```

登录到 OpenVPN 服务器后，在 bash 终端中使用 `apt` 安装 OpenVPN：

```
$ `sudo apt install openvpn -y`

```

你还需要在 OpenVPN 服务器和证书服务器上安装 EasyRSA。也通过 `apt` 安装最新版本：

```
$ `sudo apt install easy-rsa -y`

```

确保在两个 Ubuntu 服务器上都安装此软件。默认情况下，EasyRSA 将被安装到 */usr/share/easy-rsa/* 目录下。

### 设置证书颁发机构

接下来，你必须配置并构建证书服务器，使其充当 CA。最简单的方法是复制 EasyRSA 提供的模板，然后修改其配置以适应你的需求。之后，你可以初始化 PKI，构建 CA，并生成其公钥证书和私钥。

在证书服务器上导航到 *easy-rsa* 文件夹，然后创建 *vars.example* 文件的副本，命名为 *vars*：

```
$ `cd /usr/share/easy-rsa/`
$ `sudo cp vars.example vars`

```

请记住，大多数情况下，当 bash 中的命令成功运行时，屏幕上不会有输出，你将返回到提示符。

在文本编辑器中打开生成的 *vars* 文件：

```
$ `sudo nano vars`

```

在文件中，找到包含证书将由此服务器生成的组织信息的 *组织字段*；例如：

```
`--snip--`
#set_var EASYRSA_REQ_COUNTRY    "US"
#set_var EASYRSA_REQ_PROVINCE   "California"
#set_var EASYRSA_REQ_CITY       "San Francisco"
#set_var EASYRSA_REQ_ORG        "Copyleft Certificate Co"
#set_var EASYRSA_REQ_EMAIL      "me@example.net"
#set_var EASYRSA_REQ_OU         "My Organizational Unit"
`--snip--`

```

文件中的每一行默认都是注释，因此在文件运行时不会被读取或解释；它们将被忽略或抑制。删除每行开头的哈希符号（`#`），确保在调用此文件时它们能被读取。根据你的组织或个人网络，修改右侧引号中的值。这些值可以是任何你选择的内容，但不能为空。以下是一个示例：

```
`--snip--`
set_var EASYRSA_REQ_COUNTRY    "AU"
set_var EASYRSA_REQ_PROVINCE   "Queensland"
set_var EASYRSA_REQ_CITY       "Brisbane"
set_var EASYRSA_REQ_ORG        "Smithco"
set_var EASYRSA_REQ_EMAIL      "john@smithco.net"
set_var EASYRSA_REQ_OU         "Cyber Unit"
`--snip--`

```

保存并关闭文件。在 *easy-rsa* 文件夹（它应该仍然是你当前的工作目录）内执行 `easyrsa` 脚本，初始化 PKI，然后使用相同的 `easyrsa` 脚本构建 CA，这将生成 CA 的公钥证书 (*ca.crt*) 和私钥 (*ca.key*)：

```
$ `sudo ./easyrsa init-pki`
`--snip--`
Your newly created PKI dir is: /usr/share/easy-rsa/pki
$ `sudo ./easyrsa build-ca nopass`
`--snip--`
CA creation complete and you may now import and sign cert requests.
Your new CA certificate file for publishing is at:
/usr/share/easy-rsa/pki/ca.crt

```

当系统提示输入服务器的通用名称时，你可以输入任何字符串，但通常使用服务器的主机名更方便，或者按回车接受默认的通用名称。输出将包含指向你的 PKI 目录和 *ca.crt* 文件的路径；*ca.key* 文件将位于同一位置的 *private* 文件夹内。`nopass` 选项可以避免每次查询 CA 时都需要输入密码。

到此为止，CA 服务器的配置已完成。下一组配置步骤将在 OpenVPN 服务器上进行。

### 创建 OpenVPN 服务器证书和密钥

每个你计划连接到 VPN 的客户端都需要自己的公钥证书和私钥文件。这些文件允许证书服务器、VPN 服务器以及 VPN 上的任何其他客户端对客户端进行认证，并允许所有设备之间进行通信。VPN 服务器也需要自己的证书和密钥，原因相同。本部分内容描述了如何为 OpenVPN 服务器签署证书并生成密钥。连接客户端到 OpenVPN 服务器的过程将类似。

在 OpenVPN 服务器上，导航到*easy-rsa*文件夹，并按照之前的方式初始化该服务器的 PKI：

```
$ `cd /usr/share/easy-rsa`
$ `sudo ./easyrsa init-pki`

```

就像每个连接到 VPN 的客户端需要一个证书和密钥一样，OpenVPN 服务器本身也需要一个由 CA 签署的证书。为此，首先从 OpenVPN 服务器生成一个证书请求：

```
$ `sudo ./easyrsa gen-req server nopass`
Using SSL: openssl OpenSSL 1.1.1f 31 Mar 2020
Generating a RSA private key
.................................+++++
........................................+++++
writing new private key to '/usr/share/easy-rsa/pki/private/server.key.2ljAQtgUYY'
-----
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Common Name (eg: your user, host, or server name) [server]:
Keypair and certificate request completed. Your files are:
req: /usr/share/easy-rsa/pki/reqs/server.req
key: /usr/share/easy-rsa/pki/private/server.key

```

当系统提示时，按回车键接受 VPN 服务器的默认公用名称 `server`，或者提供自定义名称。输出显示已生成一个 RSA 私钥，并指示脚本将生成的服务器密钥和证书请求存储的位置。

将生成的*server.key*文件复制到 VPN 服务器的 OpenVPN 配置目录：

```
$ `sudo cp /usr/share/easy-rsa/pki/private/server.key /etc/openvpn/`

```

使用`rsync`将*server.req*文件复制到证书服务器，替换其中的用户和 CA-ip 占位符，使用你证书服务器的相关用户名和 IP 地址：

```
$ `sudo rsync -ruhP /usr/share/easy-rsa/pki/reqs/server.req` `user``@``CA``_``ip``:/tmp/`

```

接下来，输入以下命令以登录到你的证书服务器，然后导入并签署之前生成的 VPN 证书请求，从而使 VPN 通信加密并确保安全：

```
$ `ssh` `user``@``CA_ip`
$ `cd /usr/share/easy-rsa/`
$ `sudo ./easyrsa import-req /tmp/server.req` ❶ `server`
$ `sudo ./easyrsa sign-req` ❷ `server`

```

第一个`easyrsa import-req`命令用于导入请求。第二个参数是你之前为 VPN 服务器创建的公用名称 ❶。要签署请求，传递`easyrsa sign-req`命令时需要传入参数`server` ❷来指定请求类型，然后再次输入公用名称。（稍后，当签署客户端请求时，你将使用相同的命令，参数改为`client`。）

当系统询问是否确认详情无误时，仔细检查以确保公用名称已按预期设置，然后键入`yes`并按回车键完成导入和签署过程。你需要将生成的*server.crt*证书文件（连同 CA 证书）从 CA 服务器复制回 OpenVPN 服务器，以便彼此验证：

```
$ `sudo rsync -ruhP /usr/share/easy-rsa/pki/issued/server.crt` `user``@``vpn_ip``:/tmp/`
$ `sudo rsync -ruhP /usr/share/easy-rsa/pki/ca.crt` `user``@``vpn_ip``:/tmp/`

```

在 OpenVPN 服务器上，将相关文件移动到*/etc/openvpn/*目录：

```
$ `sudo mv /tmp/server.crt /etc/openvpn/`
$ `sudo mv /tmp/ca.crt /etc/openvpn/`

```

接下来，你需要一个 Diffie-Hellman 密钥用于设备间的密钥交换。*Diffie-Hellman 密钥交换*是一种通过公共通信通道安全地传递公钥和私钥信息的方式。如果没有此功能，就无法在像互联网这样的公共网络上创建安全的加密通道。

您还需要一个*HMAC 签名*来使过程更加安全。HMAC 签名用于 HMAC 认证并配合秘密密钥，是一种验证消息或有效载荷完整性的方法。在这个过程中使用 HMAC 签名将保持密钥交换的完整性，并允许您验证密钥的真实性。

在您的 VPN 服务器上，导航到您的*easy-rsa*目录，并使用之前创建的`easyrsa`脚本生成共享密钥：

```
$ `cd /usr/share/easy-rsa/`
$ `sudo ./easyrsa` ❶ `gen-dh`
$ `sudo` ❷ `openvpn --genkey secret ta.key`
$ `sudo cp /usr/share/easy-rsa/ta.key /etc/openvpn/`
$ `sudo cp /usr/share/easy-rsa/pki/dh.pem /etc/openvpn/`

```

`gen-dh`参数❶创建 Diffie-Hellman 密钥，这可能需要很长时间并产生大量输出。`openvpn --gen-key secret`❷命令快速生成 HMAC 签名，如果成功，您将不会看到任何输出。这些过程会创建*/usr/share/easy-rsa/ta.key*和*/usr/share/easy-rsa/pki/dh.pem*文件。将它们复制到 OpenVPN 配置目录*/etc/openvpn/*下：

```
$ `sudo cp /usr/share/easy-rsa/ta.key /etc/openvpn/`
$ `sudo cp /usr/share/easy-rsa/pki/dh.pem /etc/openvpn/`

```

此时，您已经创建了服务器所需的所有证书和密钥。

#### 创建客户端证书

接下来，您需要创建客户端证书和密钥，以便客户端能够连接到 VPN，这些证书与服务器证书相同，但与每个单独的客户端设备相关。最有效的方法是在服务器上创建所需的文件，而不是在客户端上创建，这样可以避免不必要的设备间文件传输。在 OpenVPN 服务器上，为文件创建一个安全位置：

```
$ `sudo mkdir -p /etc/openvpn/client-configs/keys/`

```

导航到*easy-rsa*目录，为客户端生成新的证书请求，将密钥复制到您刚才创建的目录，并将请求文件安全地复制到您的 CA 服务器，如下所示：

```
$ `cd /usr/share/easy-rsa/`
$ `sudo ./easyrsa gen-req` ❶ `myclient` `nopass`
$ `sudo cp /usr/share/easy-rsa/pki/private/``myclient``.key \`
`    /etc/openvpn/client-configs/keys/`
$ `sudo rsync -ruhP /usr/share/easy-rsa/pki/reqs/``myclient``.req` `user``@``CA_ip``:/tmp/`

```

系统会要求您为请求输入密码短语；请输入并确保将其保存以供以后参考。系统还会要求您为 VPN 客户端输入公共名称。这个名称需要对每个提供 VPN 访问的客户端不同，因此考虑使用客户端的主机名（例如`myclient`；将`myclient`❶更改为您选择的客户端名称）。

在您的证书服务器上，导航到*easy-rsa*目录：

```
$ `cd /usr/share/easy-rsa/`

```

使用客户端的公共名称（例如`myclient`）导入请求，然后使用`client`指令签署，而不是您之前使用的`server`指令：

```
$ `sudo ./easyrsa import-req /tmp/``myclient``.req` `myclient`
$ `sudo ./easyrsa sign-req client` `myclient`

```

确认公共名称是否正确，然后输入`yes`并按回车键完成命令。

最后，将新生成的证书安全地复制回 OpenVPN 服务器：

$ `sudo rsync -ruhP /usr/share/easy-rsa/pki/issued/myclient.crt` `user@vpn_ip``:/tmp/`

为了确保 VPN 正常工作，您之前创建的*ta.key*和*ca.crt*文件，以及*myclient.crt*文件，需位于 OpenVPN 服务器上的客户端配置目录中。在您的 VPN 服务器上，将这些文件复制到*/etc/openvpn/client-configs/keys/*目录：

```
$ `sudo cp /usr/share/easy-rsa/ta.key /etc/openvpn/client-configs/keys/`
$ `sudo mv /tmp/``myclient``.crt /etc/openvpn/client-configs/keys/`
$ `sudo cp /etc/openvpn/ca.crt /etc/openvpn/client-configs/keys/`

```

至此，您已创建了连接客户端到 OpenVPN 服务器所需的文件。您可以根据需要多次重复此过程。只需确保每次为新客户端生成文件时，都会将客户端名称从 `myclient` 更改为其他名称。

### 配置 OpenVPN

证书服务器设置完成后，您可以配置 OpenVPN 服务器。为此，您将复制一个模板 OpenVPN 配置并根据需要进行修改。

在 OpenVPN 服务器上，将配置模板复制到 OpenVPN 配置目录：

$ `sudo cp /usr/share/doc/openvpn/examples/sample-config-files/server.conf /etc/openvpn/`

使用文本编辑器打开生成的 *server.conf* 文件（本示例使用 nano）：

```
$ `sudo nano /etc/openvpn/server.conf`

```

与任何配置文件一样，打开它并熟悉其内容。您可能会注意到，这些配置文件使用 `#` 和 `;` 来标记注释行。

一旦您对可用选项感到熟悉，您可能会决定更改 VPN 使用的端口或协议。找到以 `port` 或 `proto` 开头的行，注意分号用于注释掉未激活的行：

```
`--snip--`
port 1194
`--snip--`
`;`proto tcp
proto udp
`--snip--`

```

OpenVPN 可以通过 UDP 或 TCP 运行，但默认使用 UDP，且默认端口为 1194\。不过，您可以让它运行在任何您喜欢的端口上，但如果您做了更改，您需要在后续的命令或文件中也做相应的更改。另外，确保文件中提到的证书和密钥与您在前面的章节中配置的一致：

```
`--snip--`
ca `ca.crt`
cert `server.crt`
key `server.key`
`--snip--`

```

当您到达 Diffie-Hellman 部分时，确保文件与您之前创建的文件匹配；配置文件默认列出 *dh2048.pem*，但您需要将其更改为 *dh.pem*：

```
`--snip--`
`#`dh dh2048.pem
`dh dh.pem`
`--snip--`

```

此外，`redirect-gateway` 和 `dhcp-option` DNS 指令不应被注释掉，因此请删除这些行开头的分号：

```
`--snip--`
push "redirect-gateway def1 bypass-dhcp"
`--snip--`
push "dhcp-option DNS 208.67.222.222"
push "dhcp-option DNS 208.67.220.220"
`--snip--`

```

这些指令确保所有流量都会通过 VPN 而不是不安全的互联网。您可以保留 DNS 的默认设置，或者设置为您希望使用的任何 DNS 服务器，如 Quad9 (*9.9.9.9*)、Google (*8.8.8.8*)，或者如果您有配置的 Pi-Hole DNS 服务器，可以设置为其地址，正如在 第七章 中描述的那样。

接下来，检查 `tls-auth` 指令是否设置为 `0`，并确保没有用分号注释掉，同时确认 `cipher` 设置为 `AES-256-CBC`。然后，在 `cipher` 指令后立即添加一个 `auth` 指令：

```
`--snip--`
tls-auth ta.key 0
`--snip--`
cipher AES-256-CBC
`auth SHA256`
`--snip--`

```

`tls-auth` 指令确保您之前配置的 HMAC 签名确实会被用于保护 VPN。此处提供了多种加密算法可供选择，AES-256 是一个合理的选择，因为它的加密效果良好且被广泛支持。`SHA256` 表示用于 HMAC 消息摘要的算法，这意味着计算出的哈希值将是一个 SHA256 哈希，被认为是安全的，并且比其他一些哈希算法更不容易发生哈希冲突。

为了使 VPN 更安全，移除`user`和`group`指令中的分号，这样 VPN 服务将以较少的权限运行，并理想地减轻权限提升攻击的风险：

```
`--snip--`
user nobody
group nogroup
`--snip--`

```

完成这些更改后，保存并关闭配置文件。

OpenVPN 配置已经完成，但你需要对服务器的网络设置进行一些更改。首先，你必须配置 IP 转发，否则 VPN 不会处理接收到的流量：

```
$ `sudo sysctl -w net.ipv4.ip_forward=1`

```

重新加载`sysctl`以使更改生效，如下所示。

```
$ `sudo sysctl -p`
net.ipv4.ip_forward = 1

```

该命令可能会输出在*sysctl.conf*文件中修改的行。

#### 配置防火墙

这个过程的第一步是找到你 VPN 服务器的公共网络接口；你的服务器可能有多个网络接口，选择错误的接口会导致 VPN 无法正确地将流量路由到互联网：

```
$ `ip route | grep -i default`
default via 202.182.98.1 dev `ens3` proto dhcp src 202.182.98.40 metric 100

```

在这个输出中，网络接口被称为`ens3`，但你的接口可能会不同。`ip route`显示的*默认路由*将是你主机的公共网络接口。你需要这个信息来正确配置防火墙。

在大多数防火墙中，你设置规则的顺序是最重要的考虑因素。在 UFW 中，规则是按照以下顺序从规则文件中评估的：首先是*before.rules*，然后是*user.rules*，最后是*after.rules*。防火墙必须正确识别并通过 VPN 流量，因此规则需要放在防火墙配置的顶部。要在 UFW 中做到这一点，打开*before.rules*文件进行编辑：

```
$ `sudo nano /etc/ufw/before.rules`

```

然后，在文件顶部添加这些行，以允许通过你在前述命令中识别的公共网络接口传递 OpenVPN 客户端流量：

```
*nat
:POSTROUTING ACCEPT [0:0]
-A POSTROUTING -s `10.8.0.0/24`-o `ens3` -j MASQUERADE
COMMIT

```

网络*10.8.0.0/24*表示连接到你 VPN 的客户端将被分配的地址。这些地址应该与网络中使用的地址不同。如果你的网络使用*192.168.1.x*地址，就不要在 VPN 网络中使用*192.168.1.x*地址。只要你的网络使用的地址不是*10.8.0.x*，先前的配置就是安全的。

保存并关闭文件。UFW 还需要接受转发的数据包，而不是丢弃它们。你可以通过修改 UFW 配置文件来允许这一点：

```
$ `sudo ufw default allow FORWARD`

```

最后，防火墙需要允许 VPN 使用的端口和协议发送和接收流量，以及用于服务器管理的 SSH。根据你在*etc/openvpn/server.conf*中设置的配置，输入以下命令以允许正确的端口和协议：

```
$ `sudo ufw allow 1194/udp`

```

接下来，允许 OpenSSH：

```
$ `sudo ufw allow OpenSSH`

```

重启防火墙以使更改永久生效：

```
$ `sudo ufw disable`
$ `sudo ufw enable`

```

在防火墙重启时，你的 SSH 连接可能会中断，你可能需要重新登录。

#### 启动 VPN

到此，你准备好启动 VPN 了。使用`systemctl`来启动 VPN，这是一个用于控制 Ubuntu 服务的工具，传递你的服务器的公共名称：

```
$ `sudo systemctl start openvpn@``server`

```

检查 VPN 的状态：

```
$ `sudo systemctl status openvpn@``server`

```

如果正常工作，输出应该显示`active (running)`。

按 Q 返回终端，然后设置 VPN 在服务器启动时自动启动：

```
$ `sudo systemctl enable openvpn@``server`

```

你的 VPN 现在应该已经启动并运行，并准备好接收客户端连接。

#### 配置 VPN 客户端

客户端必须配置*.ovpn*文件才能连接到 VPN 服务器，并通过安全隧道发送和接收流量。如果你有多个客户端需要连接，创建这些配置文件可能会很繁琐，因此我们将使用一个容易重复的程序来为我们完成这项工作。我们将在 OpenVPN 服务器上生成配置文件，然后将这些配置文件传输到相关客户端。

在你的 OpenVPN 服务器上，为客户端配置文件创建一个安全的位置（如*/etc/openvpn/client-configs/files/*），复制 OpenVPN 提供的另一个模板，并用文本编辑器打开生成的*base.conf*文件：

```
$ `sudo mkdir -p /etc/openvpn/client-configs/files/`
$ `sudo cp /usr/share/doc/openvpn/examples/sample-config-files/client.conf \`
`    /etc/openvpn/client-configs/base.conf`
$ `nano /etc/openvpn/client-configs/base.conf`

```

熟悉文件内容。如果你在前面的步骤中更改了端口或协议，请在此文件中进行相同的更改。

```
`--snip--`
;proto tcp
`proto udp`
`--snip--`
remote `vpn_ip` `1194`
;remote `vpn_ip` `1194`
`--snip--`

```

同时，取消注释`user`和`group`指令：

```
`--snip--`
user nobody
group nogroup
`--snip--`

```

注释掉 SSL/TLS 参数：

```
`--snip--`
`#`ca ca.crt
`#`cert client.crt
`#`key client.key
`--snip--`

```

注释掉`tls-auth`指令：

```
`--snip--`
`#`tls-auth ta.key 1
`--snip--`

```

将`cipher`和`auth`指令设置为在其他配置文件中找到的值：

```
`--snip--`
`cipher AES-256-CBC`
`auth SHA256`
`--snip--`

```

最后，在文件末尾添加以下内容：

```
`--snip--`
key-direction 1

```

`key-direction`指令指示客户端在客户端-服务器对中哪一方将提供密钥，因此为 VPN 隧道提供加密。该值可以设置为`0`或`1`，但此配置应设置为`1`，因为这将通过强制客户端-服务器和服务器-客户端通信使用不同的密钥来提供更好的整体安全性。保存并关闭文件。

你可以通过编写并执行脚本来轻松创建客户端配置，将所有这些元素整合在一起。创建一个*.sh*文件来放置你的脚本，赋予它可执行权限，然后用文本编辑器打开它（此例中使用 nano）：

```
$ `sudo touch /etc/openvpn/client-configs/client_config.sh`
$ `sudo chmod +x /etc/openvpn/client-configs/client_config.sh`
$ `sudo nano /etc/openvpn/client-configs/client_config.sh`

```

将列表 5-1 中的脚本复制到文件中。

```
#!/bin/bash
KEY_DIR=/etc/openvpn/client-configs/keys
OUTPUT_DIR=/etc/openvpn/client-configs/files
BASE_CONFIG=/etc/openvpn/client-configs/base.conf
cat ${BASE_CONFIG} \
    <(echo -e '<ca>') ${KEY_DIR}/ca.crt \
    <(echo -e '</ca>\n<cert>') ${KEY_DIR}/${1}.crt \
    <(echo -e '</cert>\n<key>') ${KEY_DIR}/${1}.key \
    <(echo -e '</key>\n<tls-auth>') ${KEY_DIR}/ta.key \
    <(echo -e '</tls-auth>') > $ {OUTPUT_DIR}/${1}.ovpn

```

列表 5-1：用于生成客户端配置（*.ovpn*）文件的脚本

保存并关闭文件。第一行告诉 bash，后续内容是脚本。接下来的三行是变量，如果你的密钥目录、输出目录或基础配置文件和文件夹与本章中的示例不同，可以在此处修改。

按照列表 5-2 中所示，在*client-configs*目录中执行脚本，客户端名称作为唯一参数。客户端名称应与之前步骤中创建的证书和密钥文件中的名称匹配。要为其他客户端生成配置文件，请确保生成它们的证书和密钥，然后使用这些文件通过列表 5-1 中的脚本为该客户端创建相应的*.ovpn*文件。别忘了，这涉及到创建证书请求、将其传输到证书服务器签名，然后将其传输回 VPN 服务器，放置在*client-configs*目录中。

列表 5-2 展示了对`myclient`客户端执行脚本，并列出了生成文件的命令。

```
$ `cd /etc/openvpn/client-configs/`
$ `sudo ./client_config.sh` `myclient`
$ `ls -lah /etc/openvpn/client-configs/files/`
total 20
drwxrwxr-x 2 test test 4096 Apr 28 23:22 ./
drwxrwxr-x 4 test test 4096 Apr 28 23:21 ../
-rw-rw-r-- 1 test test 11842 Apr 28 23:22 `myclient``.ovpn`

```

列表 5-2：执行来自列表 5-1 的脚本

一旦为该客户端创建了*.ovpn*文件，通过 rsync 将文件下载到本地计算机，然后将其导入到该设备的 OpenVPN 客户端中。

```
$ `rsync -ruhP` `user@vpn_ip:``/etc/openvpn/client-configs/files/``myclient``.ovpn ./`

```

OpenVPN 为大多数操作系统提供客户端应用程序，包括 Windows、Linux、macOS、iOS 和 Android。你可以在 OpenVPN 官网找到这些应用：[`openvpn.net/community-downloads/`](https://openvpn.net/community-downloads/)。

完成后，你现在可以导入*.ovpn*配置文件，连接到 VPN，并以更私密、更安全的方式上网。如果你计划使用 Linux 客户端连接到 VPN，可以使用以下命令安装 OpenVPN：

```
$ `sudo apt install openvpn -y`

```

然后，使用你的配置文件和以下命令连接到 VPN：

```
$ `sudo openvpn` `myclient``.ovpn`

```

请参阅“测试你的 VPN”在第 89 页了解更多可以确保 VPN 安全性的测试方法。

#20: 使用 Wireguard 创建 VPN

现代版本的 Ubuntu（从 2020 年 3 月开始的版本）已经将 Wireguard 集成到内核中，因此安装和启动非常简单。此时，Wireguard 并未集成到许多网络硬件中，因此你需要手动配置每个端点来连接到它，而不是像配置路由器并通过 VPN 隧道传递所有网络流量那样简单。在这个项目中，你将按照创建云端虚拟机的说明，创建一个 Wireguard 服务器，然后安装和配置 Wireguard。我们将为服务器及任何客户端创建相应的公钥和私钥对，按需配置服务器防火墙，配置并连接客户端，并测试 VPN 以确保它正常工作。只要你连接到 Wireguard VPN，互联网流量就会变得安全可靠。

### 安装 Wireguard

使用第一章中的项目 3 提供的说明创建一个新的 Ubuntu 服务器。通过 SSH 以标准用户（非 root 用户）登录服务器：

```
$ `ssh` `user@your_server_IP`

```

然后，使用`apt`安装 Wireguard，并指定`-y`跳过确认提示：

```
$ `sudo apt install wireguard -y`

```

接下来，您将创建连接和加密 VPN 所需的公钥和私钥。

### 设置密钥对

由于您即将创建的文件或文件夹具有敏感性，建议比平常更加严格地设置权限。您可以运行以下命令，确保只有文件的所有者可以读取和写入该文件：

```
$ `umask 077`

```

这个`umask`命令在退出终端会话后不会持续有效，但仅允许文件和文件夹的所有者在此会话中读取和写入您创建的文件和文件夹。

现在，使用`wg genkey`命令创建私有的 Wireguard 密钥：

```
$ `wg genkey | sudo tee /etc/wireguard/private.key`

```

终端中显示的输出是您的私钥，它将存储在命令指定的*private.key*文件中。请勿共享此密钥。像对待密码一样对待它——它是您 VPN 安全的保障。

创建私钥后，您需要一个相应的公钥提供给您的客户端，以便他们可以进行服务器身份验证：

$ `sudo cat /etc/wireguard/private.key | wg pubkey | sudo tee /etc/wireguard/public.key`

该命令首先使用`cat`读取*private.key*文件的内容。然后，`wg pubkey`命令使用私钥生成公钥。公钥随后输出到终端并保存到*public.key*文件中。

现在您已经拥有公钥/私钥对，可以配置您的 VPN 服务器和客户端。

### 配置 Wireguard

Wireguard 需要一个配置文件才能运行。安装 Wireguard 时并不会创建此文件，因此您需要从头开始创建一个。使用文本编辑器创建并打开*/etc/wireguard/wg0.conf*文件：

```
$ `sudo nano /etc/wireguard/wg0.conf`

```

将以下内容添加到文件中：

```
`[Interface]`
`PrivateKey =` `your_private_key`
`Address =` `10.8.0.1/24`
`ListenPort =` `26535`
`SaveConfig = true`

```

将*your_private_key*替换为您之前创建的私钥。您的密钥将是*/etc/wireguard/private.key*文件的内容。地址将是您希望 VPN 客户端连接到服务器时分配的子网地址；确保该子网与您的私有网络不同。例如，如果您在网络中使用*192.168.1.x*地址，则避免在 VPN 中使用*192.168.1.x*地址。监听端口应该是 1025 到 65535 之间的任意端口，随机选择。这个端口是您的服务器和客户端用来通信的端口。完成后，保存并退出配置文件。

此时，服务器的网络设置需要一些修改。使用以下命令配置 IP 转发，使 VPN 能够转发其接收到的流量，然后重启`sysctl`以使更改生效：

```
$ `sudo sysctl -w net.ipv4.ip_forward=1`
$ `sudo sysctl -p`

```

接下来，你需要配置防火墙以允许 VPN 流量进入和离开服务器。

#### 配置防火墙

在本节中，我们将讨论 *简单防火墙（UFW）* 的使用，这是 Ubuntu 内置的防火墙，旨在简化防火墙配置。要配置防火墙，首先需要识别 VPN 的正确网络接口。指定错误的接口将导致 VPN 无法正常工作。输入以下命令来查找服务器的默认网络接口：

```
$ `ip route | grep -i default`
default via 172.16.90.1 dev `ens33` proto dhcp metric 100

```

在此输出中，网络接口名为 `ens33`（你的可能不同）。`ip route` 显示的 *默认路由* 将是你主机的公共网络接口。你需要这个信息来正确配置防火墙。

接下来，通过再次使用文本编辑器打开 */etc/wireguard/wg0.conf* 文件，并将 `ens33` 替换为你的网络接口名，向 Wireguard 配置文件的底部添加以下规则：

```
$ `sudo nano /etc/wireguard/wg0.conf`
`--snip--`
SaveConfig = true
`PostUp = ufw route allow in on wg0 out on` `ens33`
`PostUp = iptables -t nat -I POSTROUTING -o` `ens33` `-j MASQUERADE`
`PreDown = ufw route delete allow in on wg0 out on` `ens33`
`PreDown = iptables -t nat -D POSTROUTING -o` `ens33` `-j MASQUERADE`

```

保存并关闭文件。这允许 Wireguard 在启动后和停止前修改防火墙配置，以便 VPN 正常工作。

此外，你需要允许通过你在本章早些时候配置的监听端口（示例中的端口为 26535）进行流量传输：

```
$ `sudo ufw allow` `26535``/udp`

```

接下来，允许 OpenSSH：

```
$ `sudo ufw allow ssh`

```

最后，更新规则后，你需要禁用并启用 UFW 以重新加载规则（你的 SSH 会话可能会被中断，你可能需要重新登录）：

```
$ `sudo ufw disable`
$ `sudo ufw enable`

```

至此，你的防火墙配置已经完成。

#### 识别 DNS 服务器

为了确保你的互联网流量安全，VPN 需要正确配置 DNS，以防止 DNS 泄漏，因为这可能会危及你的安全。为了解决这个问题，你将强制 Wireguard VPN 使用 Wireguard 服务器本身使用的 DNS。通过以下命令识别该 DNS 服务器：

```
$ `resolvectl dns ens33`

```

结果输出是你稍后在此项目中将提供给客户端的 DNS 地址——记下来。

#### 启动 VPN

理想情况下，VPN 应该在服务器启动时自动启动并准备好接受客户端连接。你可以通过使用 `systemctl` 创建并启动 Wireguard 系统服务来实现这一点：

```
$ `sudo systemctl enable wg-quick@wg0.service`
$ `sudo systemctl start wg-quick@wg0.service`

```

完成后，检查状态以确保 Wireguard 正在运行：

```
$ `sudo systemctl status wg-quick@wg0.service`

```

如果正常工作，输出应该显示 `active`。如果服务未激活或状态为失败，仔细检查配置文件和防火墙状态，确保配置中没有拼写错误或其他问题。

#### 配置 VPN 客户端

Wireguard 为 Windows、macOS、Android 和 iOS 提供了官方客户端应用程序——它们的设置过程大致相同。Linux 客户端的设置稍微复杂一些，但如果你已经成功配置了 Wireguard 服务器，配置 Linux 客户端应该非常熟悉。

#### Windows、macOS、Android 或 iOS 客户端配置

要在任何这些操作系统上配置客户端，请按照以下步骤操作：

1.  1\. 从 [`www.wireguard.com/install/`](https://www.wireguard.com/install/) 下载并安装相关客户端程序。

1.  2\. 在客户端界面中，点击 **+** 或 **添加隧道▸添加空隧道** 来从头创建一个新的 VPN 配置文件。

1.  3\. 请注意，客户端的公钥和私钥将显示出来。

1.  4\. 在名称字段中提供一个友好的名称。

1.  5\. 忽略任何按需设置或勾选框。

1.  6\. 在配置文件中添加以下详细信息，位于客户端自动生成的 PrivateKey 下方：

    ```
    `--snip--`
    `Address =` `10.8.0.2`
    `DNS =` `108.61.10.10`
    `[Peer]`
    `PublicKey =` `server_public_key`
    `AllowedIPs =` `0.0.0.0/0`
    `Endpoint =` `server_public_ip:listening_port`

    ```

    `Address` 是您希望客户端在 VPN 子网中使用的 IP 地址，每个 VPN 客户端的地址应该不同。`DNS` 应该是您在 “识别 DNS 服务器”中确定的 DNS 服务器的 IP 地址，详见 第 85 页。`PublicKey` 是您在之前的过程中为 Wireguard 服务器创建的公钥。`AllowedIPs` 是用于 *分割隧道* 的设置；列在此指令中的网络或地址之间的流量将通过 VPN 隧道传输，其他所有流量则会直接出去，绕过 VPN。将其设置为 `0.0.0.0/0` 将使所有来自客户端的流量都通过 VPN 传输。`Endpoint` 是您的 VPN 服务器的公共 IP 地址，后跟您之前指定的监听端口（在示例中为 26535）。

1.  7\. 保存配置。

1.  8\. 在 Wireguard 服务器上停止 Wireguard 服务，注意此操作会导致当前连接的用户出现停机，使用以下命令：

    ```
    $ `sudo systemctl stop wg-quick@wg0.service`

    ```

1.  9\. 使用文本编辑器打开 */etc/wireguard/wg0.conf* 配置文件：

    ```
    $ `sudo nano /etc/wireguard/wg0.conf`

    ```

1.  10\. 将客户端的详细信息添加到配置文件的底部，请记住，每个添加的对等端都需要在此文件中添加自己的 [Peer] 部分：

    ```
    `--snip--`
    `[Peer]`
    `PublicKey =` `client_public_key`
    `AllowedIPs =` `10.8.0.2`

    ```

    这个 `PublicKey` 实例是由客户端应用程序为您的 Wireguard 客户端创建的公钥。在文件的 `[Peer]` 部分，`AllowedIPs` 指定了允许通过 VPN 隧道发送流量的 IP 地址。将其设置为您希望客户端在 VPN 网络上使用的特定主机 IP，必须与您在客户端配置中为此对等端配置的 IP 相匹配。

1.  11\. 保存并关闭文件。

1.  12\. 启动 Wireguard 服务，并再次检查状态是否为 `active`：

    ```
    $ `sudo systemctl start wg-quick@wg0.service`
    $ `sudo systemctl status wg-quick@wg0.service`

    ```

    返回您的客户端，激活 VPN 连接。成功连接后，ping 您的 Wireguard 服务器的 VPN 地址（例如 10.8.0.1）：

    ```
    $ `ping 10.8.0.1`
    PING 10.8.0.1 (10.8.0.1): 56 data bytes
    64 bytes from 10.8.0.1: icmp_seq=0 ttl=57 time=43.969 ms
    64 bytes from 10.8.0.1: icmp_seq=0 ttl=57 time=43.969 ms
    64 bytes from 10.8.0.1: icmp_seq=0 ttl=57 time=43.969 ms
    64 bytes from 10.8.0.1: icmp_seq=0 ttl=57 time=43.969 ms
    --- 10.8.0.1 ping statistics ---
    4 packets transmitted, 4 packets received, 0.0% packet loss
    round-trip min/avg/max/stddev = 43.969/43.969/43.969/0 ms

    ```

    成功的结果表示您的 VPN 连接在客户端和服务器之间工作正常。对任何额外的客户端，重复此过程。

#### Linux 客户端

配置 Linux 客户端，按照以下步骤进行操作：

1.  1\. 安装 Wireguard 和 resolvconf（用于 DNS 配置）：

    ```
    $ `sudo apt install wireguard resolvconf -y`

    ```

1.  2\. 为客户端生成公钥/私钥对：

    ```
    $ `wg genkey | sudo tee /etc/wireguard/private.key`
    $ `sudo cat /etc/wireguard/private.key | wg pubkey | sudo tee \`
    `     /etc/wireguard/public.key`

    ```

1.  3\. 创建 Wireguard 客户端配置文件：

    ```
    $ `sudo nano /etc/wireguard/wg0.conf`
    `[Interface]`
    `PrivateKey =` `client_private_key`
    `Address =` `10.8.0.3`
    `DNS =` `108.61.10.10`
    `[Peer]`
    `PublicKey =` `server_public_key`
    `AllowedIPs =` `0.0.0.0/0`
    `Endpoint =` `server_public_ip:listening_port`

    ```

1.  4\. 保存并关闭文件。

1.  5\. 在 Wireguard 服务器上停止 Wireguard 服务：

    ```
    $ `sudo systemctl stop wg-quick@wg0.service`

    ```

1.  6\. 使用文本编辑器打开 */etc/wireguard/wg0.conf* 配置文件：

    ```
    $ `sudo nano /etc/wireguard/wg0.conf`

    ```

1.  7\. 将客户端详细信息添加到配置文件的底部：

    ```
    `--snip--`
    `[Peer]`
    `PublicKey =` `client_public_key`
    `AllowedIPs =` `10.8.0.3`

    ```

    此处的 `PublicKey` 是由客户端应用程序为你的 Wireguard 客户端生成的公钥。在文件的 `[Peer]` 部分，`AllowedIPs` 指的是允许通过 VPN 隧道发送流量的 IP 地址。将其设置为你希望客户端在 VPN 网络中拥有的特定主机 IP。

1.  8\. 保存并关闭文件。

1.  9\. 启动 Wireguard 服务并仔细检查状态是否为“活动”：

    ```
    $ `sudo systemctl start wg-quick@wg0.service`
    $ `sudo systemctl status wg-quick@wg0.service`

    ```

    返回客户端，使用以下命令激活 VPN 连接：

    ```
    $ `wg-quick up wg0`

    ```

    成功连接后，ping 你的 Wireguard 服务器的 VPN 地址（例如 10.8.0.1）。如果测试成功，表示你的 VPN 连接在客户端和服务器之间正常工作。要断开 Linux 客户端与 VPN 服务器的连接，可以使用以下命令：

    ```
    $ `wg-quick down wg0`

    ```

    对任何其他要添加的客户端重复此过程。

## 测试你的 VPN

无论你选择了哪个 VPN，都可以通过像 [`www.whatismyip.com/`](https://www.whatismyip.com/) 这样的网站，在未连接 VPN 时找到你的公共 IP 地址。完成后，连接到 VPN 并刷新页面。此时，你的公共 IP 地址应该变成 VPN 服务器的 IP 地址。另一个测试 VPN 的方法是使用像 [`dnsleaktest.com/`](https://dnsleaktest.com/) 这样的服务。进行标准测试后，你应该能清楚地看到 VPN 配置是否存在问题。如果你的实际公共 IP 地址被隐藏，并且 DNS 泄漏测试只显示了你配置的 DNS 服务器，那么说明你已成功搭建了自己的私人 VPN 服务器。

## 总结

将多个客户端连接到你的 OpenVPN 或 Wireguard 服务器将允许它们之间的流量像在同一网络上一样传递。这意味着你可以轻松地通过同时将多个设备连接到你的 VPN，来远程管理这些设备。本章讲解了如何搭建你自己的私人 VPN，使用 OpenVPN 或更加轻量、快速的 Wireguard，给你完全的控制权。通过连接到 VPN，你的私人互联网流量将变得真正安全和私密。
