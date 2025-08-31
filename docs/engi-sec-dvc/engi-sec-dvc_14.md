## 第十一章：访问控制与管理**

![Image](img/common.jpg)

在 IT 系统中，用户和权限的管理有着悠久的历史，因为从一开始，人类就在其中扮演了重要角色。相比之下，嵌入式系统过去并不打算进行交互，通常只有少数用户，甚至只有一个系统用户。

今天，随着物联网商业模型和应用场景的日益复杂，许多参与者都涉及到物联网设备的生命周期过程——从开发者、维护人员、第三方服务提供商到最终用户。设备必须能够*处理*这些不同的角色，并将它们*分离*开来。此外，设备上运行的内部过程和应用程序也需要各种权限来完成其目的。按照最小权限原则限制这些权限，可以避免设备遭受严重损害。

本章将从多个角度讨论访问控制如何对设备安全做出重要贡献。接下来，我将介绍你可以用来在运行 Linux 的设备上实现访问限制的常见概念。章末的案例研究则展示了如何使用 AppArmor 工具对进程进行限制的实际可能性。

### **日常威胁**

数据库泄露和漏洞几乎每天都会发生。犯罪分子发布或出售的数据往往包含大量的用户名和密码。你的物联网设备的凭证可能也在这些泄露的秘密中，使得攻击者能够登录到你的产品。你可能会得出结论，认为这些是客户的风险，并且在凭证被盗的情况下，你不需要承担任何责任，但这并不完全正确。

你至少需要问自己两个问题：“我们是否将最终用户账户与制造商账户分开？”以及“我们是否尽可能限制这些最终用户账户，以便在凭证泄露时控制损害？”如果你不考虑这些问题，它们可能会反弹，至少会对你产品的声誉产生负面影响。

渗透测试人员、安全研究人员，甚至是客户，都会定期发现产品中的未知漏洞。随着设备和软件复杂度的增加，以及网络接口上暴露的服务数量的增加，安全问题的概率也会增加。即使你已经建立了一个稳健的漏洞管理流程，如第一章所述，并且准备好以安全的方式发布更新，如第九章所解释的那样，你的产品在某些时间段内仍然可能会有漏洞。如果这种情况导致在实际应用中发生攻击，严重影响设备，专家们自然会问：“为什么这个有漏洞的应用程序在其目的相对有限的情况下，竟然能够影响系统的所有部分？”

当前，多个行业的制造商普遍讨论的一个话题是将他们的设备转变为*平台*，可以从相应的市场安装并运行应用程序，而这些市场又由全球的应用程序开发者提供支持。同样，可能看起来显而易见的是，这些开发者负责其应用程序的安全。然而，如果应用程序中实际存在的漏洞被利用，那么保护系统进程和配置以及其他应用免受问题应用影响的责任将落在平台设计者身上。

有时，访问控制只在软件或操作系统层面上被考虑。然而，对于嵌入式系统来说，另一个额外的威胁具有重要的相关性：物理访问。攻击者以及渗透测试人员和研究人员可以通过与本地（调试）接口如 JTAG、通用异步接收器-发射器（UART）或互连电路（I²C）总线交互来对设备进行物理分析。

通过声称“没有人会打开我们的设备”，甚至如果真有人打开，“内部结构如此复杂，就连我们的工程师都不了解所有细节”来减轻这种威胁是常见的，但这种说法很少是合理的。有兴趣的攻击者肯定会拆开产品外壳并寻找本地接口，如果他们有一定动机，他们也会愿意花时间手动进行设备反向工程，直到达到他们的目的。

### **访问控制与损害隔离**

在许多情况下，完善的访问控制管理可以减少甚至防止攻击的影响。这是一个完美的例子，展示了工程师和开发者如何严肃对待深度防御原则。如果一个安全层（如用户凭证的机密性）失败，或发现未知的软件漏洞，访问控制层将介入并防止最严重的后果。

然而，这只有在你有一个合理的基础来决定是否应当授予访问权限，以及在授予权限时，实际需要多少访问权限的情况下才有效。在授予用户访问文件或硬件资源等对象的权限时，你应该考虑几个属性。

权限可以根据用户身份授予或拒绝，这种方法有时被称为*基于身份的访问控制（IBAC）*。然而，独立地管理每个用户或人可能过于复杂。因此，*基于角色的访问控制（RBAC）*应运而生，它根据用户的角色来设定权限。这不仅简化了权限管理，而且要求你明确地将角色分配给每个用户，从而提高了透明度。

另一种访问控制方法依赖于主体、对象，甚至可能是它们的环境的属性。这种方法被称为*基于属性的访问控制（ABAC）*，它比 IBAC 或 RBAC 能做出更动态的访问决策。

优化安全性方面的访问控制的常见策略是将权限减少到给定用户或应用程序所需的最低权限。然而，这正是问题的关键，因为这些最低要求通常并不明确知道。

因此，开发人员通常倾向于慷慨地设置权限，给恶意活动或入侵留下空间。他们的理由是可以理解的，因为过度限制用户和应用程序可能导致设备无法操作。而且，情况可能更糟的是，用户的职责和应用程序的访问权限可能随着时间的推移而发生变化。因此，重要的是要尽早并全面地考虑访问控制管理，贯穿整个设备生命周期。

#### ***设计与开发阶段***

在硬件设计过程中，关于对 IC 引脚、焊盘和 PCB 上的迹线的物理访问的讨论应该已经提上议程。调试端口的模糊化或在关键部件上应用环氧树脂可能是需要考虑的解决方案。通过接触开关或弹簧，产品外壳被打开时向主处理器发出信号，也可能减少物理访问的影响。

为了提高安全性，甚至可以将导电网格结构集成到塑料外壳中，持续运行信号以检测外壳是否遭到篡改。此类机制已在支付终端中得到应用。

固件开发的重要部分是系统用户、他们的角色、目录、初始文件和相应权限的规范。即使这听起来微不足道，也要确保考虑到设备预期使用情况的整个范围。结果必须在你选择的构建系统中实现（例如 Yocto 或 Buildroot），并且在整个开发过程中应监控其正确性。

接下来，必须分析设备的所有软件应用程序和服务。一方面，必须指定每个应用程序应该在哪个用户上下文中运行。这定义了进程在运行时拥有的权限，当进程被恶意行为者接管时，这一点尤为重要。仔细考虑是否 root 总是最好的选择。

另一方面，应用程序也可以在访问控制管理中发挥积极作用。以 Web 服务器为例。这种常见的应用程序控制它为连接的客户端提供的网页。某些网页界面部分，如设备管理页面，可能比其他部分更为关键。基于操作系统用户或 Web 服务器自己的用户管理，服务器必须适当配置，只允许合法用户访问管理员权限。

当然，应用程序可能有多个“管理员”用户：一个用于网页应用管理任务的`webadmin`，一个用于 Linux 系统管理的`sysadmin`，还有一个可能用于制造商访问的`superadmin`。这些用户都有不同的权限。

#### ***生产考虑事项***

设备生产通常不被视为访问控制管理中的重要步骤，但越来越多的标准——例如欧洲的 ETSI EN 303 645 消费电子标准——要求设备避免实施全球默认密码。由于设备的固件通常是一个静态的全球性工件，因此必须在生产过程中生成并设置设备专有的密码。

不仅仅是固件需要个性化。纸张、标签或产品包装的一部分也需要标明个别密码。这个过程很大程度上依赖于设备的固件结构和生产过程。

生产过程中密码生成的替代方案是强制终端用户在首次登录时设置新的自定义密码。这种方法的优点是可以使用单一的全球固件镜像，这在生产过程中更容易处理。然而，这样做也意味着你的产品仍然会有一个可以在初始化时至少使用一次的通用默认密码——无论是用户还是攻击者都可以使用。

#### ***客户活动与报废***

在现场，客户可能希望自行创建额外的用户或更改已有账户的权限。你必须决定是否允许这样做，如果允许，应在什么范围内。允许客户自由选择授予哪些权限可能导致*权限提升*，从而使最终用户获得比原本预期更多的权限。

手动更改数百台设备的密码绝对是枯燥乏味且容易出错的。因此，现代物联网基础设施需要对资产和配置（包括用户、角色和权限）进行集中管理，以确保能够保持可管理性。对于设备制造商来说，准备设备以支持轻量级目录访问协议（LDAP）集成或类似的技术可能是合理的，以简化集中管理。然而，确保在信任远程服务器来处理设备的用户、凭证和权限时，要考虑潜在的威胁。

如前所述，应用程序应被限制为仅具有正确运行所需的最小权限。然而，随着每次固件更新，设备软件组件的功能和行为可能会发生变化。新的功能可能需要更多的权限，而严格的安全性可能会成为阻碍。同时，安全更新可能会删除那些曾经需要特定权限的软件程序，这些权限之后就不再需要了。一般来说，如果您使用的是限制软件应用程序的工具并且更新了该软件，您总是需要再次检查相关权限是否与新版本匹配。

最后但同样重要的是，您必须考虑到您的设备可能会被转售并被另一位客户使用。那位客户应该仍然能够初始化设备并重置用户和权限，但不应访问前一位所有者创建的数据。同样的情况也适用于最终停用，当好奇的垃圾桶潜伏者或废品商贩可能想要获取访问受限的数据时。应该为原始客户提供一个“清除所有私人数据”按钮，确保该按钮真正执行它所说的功能，并且新添加的用户即使没有清除数据，也不能访问其他人的数据。

### **自主访问控制**

*自主访问控制（DAC）* 是一种管理系统内对象的主题和组的访问权限的基本方法。在我们的例子中，主题是 Linux 操作系统中的用户，而对象可能是文件、目录、内存位置、进程间通信以及各种系统设备和接口。此方法是 *自主的*，因为某个对象的权限是由拥有该特定对象的用户 *自行决定* 的；*所有者* 可以将权限传递给其他用户和组。

**注意**

*在 Linux 系统中，root 用户是无所不能的。它能够覆盖所有者设置的权限。因此，“成为 root”是攻击者最具吸引力的目标之一。对于开发者而言，这意味着可能被破坏的进程（例如所有监听网络端口的应用程序）不应该以 root 身份运行！*

#### ***Linux 文件系统权限***

每个 Linux 文件系统中的文件都有一个与之关联的权限字符串。如 Listing 11-1 所示，`ls` 工具可以在每一行的开头始终打印出这个字符串。

```
# ls -l
-rw-r--r-- 1 root root    23 Apr 30 13:13 README
lrwxrwxrwx 1 root root    17 Apr 30 14:15 current_logfile -> logs/73944561.log
drwxr-xr-x 2 root root  1024 Apr 30 14:14 logs
-rw-r--r-- 1 bob  guest   12 Apr 30 17:15 my_notes.md
# ls -l logs/73944561.log
-rw-r--r-- 1 root root     9 Apr 30 12:20 73944561.log
```

*Listing 11-1: Linux 上的文件权限列表*

显眼的字符串 `root root` 表明所有文件都归 root 用户和 root 组所有，除了 *my_notes.md* 文件，它归 *guest* 组中的 *bob* 所有。此外，每行的第一个字符指定文件类型，是普通文件（`-`）、目录（`d`）还是符号链接（`l`）。块设备（`b`）和字符设备（`c`）也会在那里显示。接下来的九个字符代表每个文件的权限，它们被分为三组，每组三个字符，分别表示所有者、相应组和其他人对读取（`r`）、写入（`w`）和执行（`x`）的权限。

例如，清单 11-1 中的 *README* 文件可以被 root 用户读取和写入，但其他 root 组中的用户只能读取。读权限也被授予任何其他用户，无论其所在组如何，这由后缀的 `r--` 所示。

每当你阅读关于 Linux 访问控制管理的教程或第三方源代码时，你可能会遇到一种高效的三位数字访问权限表示法。9 位的权限字符串可以高效地写成三个八进制数字：一个表示所有者权限，一个表示组成员权限，最后一个表示其他所有人的权限。

例如，`777` 表示每个人都可以读取、写入和执行给定对象。相对的，`740` 表示所有者可以读取、写入和执行；组成员仅被允许读取；而其他任何人都没有访问权限。表 11-1 详细说明了权限到八进制数字的转换及其反向转换。

**表 11-1：** Linux 文件权限的八进制表示

| **八进制** | **二进制** | **权限** |
| --- | --- | --- |
| 0 | 000 | `---` |
| 1 | 001 | `--x` |
| 2 | 010 | `-w-` |
| 3 | 011 | `-wx` |
| 4 | 100 | `r--` |
| 5 | 101 | `r-x` |
| 6 | 110 | `rw-` |
| 7 | 111 | `rwx` |

理解文件权限是直观的，但目录的行为略有不同。在目录中，读权限使得可以列举目录内的项目。写权限授予添加、删除或重命名目录项的权限。执行权限允许用户进入目录，例如使用 `cd` 命令，访问文件或子目录。

#### ***Linux 用户和组管理***

在 Linux 中，有几个工具用于管理用户和组。最基本的命令是所有发行版都可用的，甚至在嵌入式系统上也可以使用，具体如下：

| useradd | 创建一个新用户 |
| --- | --- |
| usermod | 修改现有用户的属性 |
| userdel | 删除一个用户 |
| groupadd | 创建一个新组 |
| groupmod | 修改现有组的属性 |
| groupdel | 删除一个组 |

在进行安全设备工程时，最重要的命令是 `useradd` 和 `groupadd`，因为它们可以在创建镜像时用于实现与访问控制概念相关的用户和角色。

对于用户创建，当然需要提供新用户的名字。你可能还需要指定的其他选项包括其*用户标识符*（或*UID*）（`--uid`）、其主目录（`--home`）以及是否应该自动创建该目录（`--create-home`）。此外，你可以定义是否应该创建一个与用户名相同的用户组（`--user-group`）以及用户应该属于哪些组（`--groups`）。`--shell`选项允许指定登录后的 shell，但它也可以与`false`和`nologin`一起使用，以禁用用户登录。最后，你可以通过`--password`选项设置用户的密码（以哈希形式）。

创建组时，`groupadd`命令需要指定的主要参数是组名，如果需要，还可以指定一个相应的*组标识符（GID）*（`--gid`），以便稍后引用该特定组。

用户和组可能被标记为属于系统（`--system`），与外部的、可能是人类用户相对。此类系统账户从保留的系统范围获得 UID/GID。此外，系统用户不会过期，也不会创建主目录。

**注意**

*这些工具* useradd *和* adduser *以及* groupadd *和* addgroup *容易混淆。本节中介绍的工具是基本的、可移植的版本，而其他工具可能在交互式会话中更易于使用。*

#### ***Linux 权限管理***

在 Linux 系统中，配置文件和目录的所有者、组和权限需要使用三种常见工具：

chown    允许你将文件或目录的所有者更改为另一个所有者。例如，可以使用`chown bob web.conf`将*web.conf*文件的所有者设置为*bob*。

chgrp    指定给定文件或目录的新组。例如，*manuals*目录的组可能会通过`chgrp guest manuals`命令更改为*guest*。

chmod    操作文件和目录的权限。调用`chmod +x script.sh`为*script.sh*文件添加执行权限，而`chmod -wx script.sh`则移除写入和执行权限。两者只会影响所有者的权限。通过在权限字符串前加上`g`（组）、`o`（其他人）、`u`（用户/所有者）和`a`（所有人），你还可以指定应该影响权限字符串的哪一部分。例如，`chmod go-rwx private.key`会移除所有组成员以及除了所有者之外的任何其他人的*private.key*文件的所有访问权限。

#### ***访问控制列表***

原则上，用户和组使我们能够表示任何所需的访问控制设置。然而，在某些情况下，文件或目录只能有一个所有者和一个关联组的限制使得高效的访问控制管理变得困难。

让我举个例子，因为它的用处可能不太明显。假设你有一个名为 *internal_logs* 的目录，用来存储日志和运行时数据。这个目录中的文件由五个用户创建，所有用户都属于 *service* 组。两年后，你发布了一项新的预测性维护功能，并引入了一个名为 `predmain` 的新用户，该用户只需要对 *internal_logs/freqtrack.dat* 文件具有读取权限，并且该用户不应有写入权限——以防止在遭到入侵时造成损害。你不能将 *predmain* 添加到 *service* 组中，因为那样它将拥有过多权限，你也不能将 *predmain* 设置为 *freqtrack.dat* 的所有者，因为如果遭到入侵，攻击者将会拥有过多控制权限。

一种解决方案是使用基于 Linux 文件系统扩展文件属性（xattr）实现的访问控制列表（ACLs）。根据你的系统，必须先安装 ACL 支持，并且在挂载文件系统时需要使用 `acl` 选项，才能使用 `getfacl` 和 `setfacl` 命令行工具，分别查看和更改细粒度的权限。

### **案例研究：STM32MP157F-DK2 固件的访问控制**

在本案例研究中，我首先展示了在 Yocto 中的用户和文件初始化过程。接下来，我探讨了 Linux 对某些系统文件设置的默认权限以及背后的理由。最后，作为应用层访问控制的一个例子，我查看了 SSH 守护进程 Dropbear 的配置。

#### ***Yocto 中的用户创建和文件配置***

关于访问控制的第一个问题是，在任何 Linux 平台上，你必须明确如何处理 root 用户。尤其在开发过程中，root 用户可能经常被使用，甚至没有密码。确保在生产镜像中删除调试设置。在我的案例中，我从 ST 的 `st-image-core` 镜像中移除了 `debug-tweaks` 特性，如 列表 11-2 所示。

```
EXTRA_IMAGE_FEATURES:remove = "debug-tweaks"
inherit extrausers
ROOT_PASSWORD_HASH = "\$6\$ZsFPzdUpnha4s1lG\$8Zxzo4UhZBomryn/SJSlVq97TLy..."
EXTRA_USERS_PARAMS:append = "usermod --password '${ROOT_PASSWORD_HASH}' root;"
```

*列表 11-2：为生产准备 root 用户*

我继承了 `extrausers` 类，它允许修改现有的 root 用户（例如，用强密码保护它）。`ROOT_PASSWORD_HASH` 后面的看似神秘的字符串是 Linux 预期的用户密码哈希格式。它是通过调用 `openssl passwd -6` 命令获得的，其中 `-6` 参数表示使用基于 SHA-512 的加盐哈希。另外需要注意的是，`$` 符号在这种格式中作为分隔符，并且在 Yocto 配方中需要转义。

**注意**

*在许多情况下，完全禁用 root 登录是有意义的，除非你有令人信服的理由不这样做，以防止滥用这个强大的用户权限。*

Yocto 还提供了 `useradd` 类，以便在自定义配方中进一步配置用户和组。列表 11-3 显示了创建两个系统用户 *rservice* 和 *lservice*，以及最终用户 *admin* 和 *guest*，同时还创建了两个相应的组，并将用户添加到这些组中。

```
inherit useradd
USERADD_PACKAGES = "${PN}"

USERADD_PARAM:${PN} = "--password '\$6\$fu47IexZgSH/T6d0\$9a.LjAl0sL0K...' \
                       --home /home/rservice --system rservice; \
                       --password '\$6\$tSsINjOvlFOaVrky\$9VIgdb7.LIVG...' \
                       --home /home/lservice --system lservice; \
                       --password '\$6\$VOPFagOJM.H.ZWIh\$8lELUZpkIogC...' \
                       --uid 1300 --home /home/admin admin; \
                       --password '\$6\$CPaAzKAYqkSKW42x\$KgivNUKDqsJT...' \
                       --uid 1301 --home /home/guest guest"

GROUPADD_PARAM:${PN} = "--system service; \
                        --gid 890 endusers"

GROUPMEMS_PARAM:${PN} = "--add rservice --group service; \
                         --add lservice --group service; \
                         --add admin --group endusers; \
                         --add guest --group endusers"
```

*列表 11-3：为镜像创建用户和组*

所有用户都使用 `USERADD_PARAM` 变量初始化密码。特定的 UID 仅用于最终用户账户。`GROUPADD_PARAM` 变量用于创建新组，而 `GROUPMEMS_PARAM` 则将创建的用户添加到这些组中。

在某些情况下，你可能还希望为用户创建目录并将初始文件放入其中。在列表 11-4 中，展示了一个来自自定义配方的代码片段，作为创建用户时文件配置的简单示例，包括设置所有者和组的必要命令。

```
do_install () {
        install -d -m 770 ${D}/home/rservice
        install -d -m 740 ${D}/home/lservice
        install -d -m 500 ${D}/home/admin
        install -d -m 550 ${D}/home/guest

        install -p -m 400 administration.md ${D}/home/admin/
        install -p -m 440 README ${D}/home/guest/

        chown -R rservice ${D}/home/rservice
        chown -R lservice ${D}/home/lservice
        chown -R admin ${D}/home/admin
        chown -R guest ${D}/home/guest

        chgrp -R service ${D}/home/rservice
        chgrp -R service ${D}/home/lservice
 chgrp -R endusers ${D}/home/admin
        chgrp -R endusers ${D}/home/guest
}
```

*列表 11-4：为创建的用户提供基本文件配置*

首先，为所有用户创建主目录并设置相应的权限。例如，最终用户应该只能读取提供的数据，而不能在设备上存储自己的代码或执行它。此外，管理员信息不应对 `guest` 用户可见。

服务用户则拥有更高的权限。它们可以将自定义数据加载到自己的目录中，而 `lservice` 本地服务用户拥有最高权限，因为它甚至可以读取和写入远程服务用户的目录。

通过这些基本步骤，你可以为设备的访问控制管理打下基础。

#### ***系统文件和预定义用户的探索***

幸运的是，Linux 及其发行版已经为各种系统文件的权限设置做好了配置。我们来看看我的 STM32MP157F 设备固件中的一些具体示例。

Linux 上的用户和密码管理由 */etc/passwd* 和 */etc/shadow* 文件实现。如列表 11-5 所示，第一个文件被标记为对所有人可读，因为可能有各种合理的理由需要读取系统上的用户列表。然而，每个用户的实际密码哈希并不包含在 *passwd* 文件中。它存储在 *shadow* 文件中，该文件仅根用户可读，用于登录验证目的。

```
# ls -l /etc/passwd /etc/shadow
-rw-r--r-- 1 root root 1404 ... /etc/passwd
-r-------- 1 root root  884 ... /etc/shadow
```

*列表 11-5：Linux 上密码文件的访问权限*

*影像*密码存储已经存在数十年。其主要思想是限制非特权用户对密码哈希的访问，因为如果用户使用了弱密码，攻击者如果能够访问相应的密码哈希，就能对其进行暴力破解攻击。

**注意**

*如果你想知道如果* /etc/shadow *只有根用户可读，甚至连 root 都不能修改密码，答案是：超级用户 root 具有类似 Chuck Norris 的能力；它甚至可以写入只读文件。*

列表 11-6 展示了我设备的 microSD 卡（*/dev/mmcblk0*）、Linux RNG 设备（*/dev/urandom*）和 STM32MP157F 的硬件 RNG 设备（*/dev/hwrng*）的权限字符串。

```
# ls -l /dev/mmcblk0
brw-rw---- 1 root disk 179, 0 Apr 28 17:42 /dev/mmcblk0 # ls -l /dev/urandom /dev/hwrng
crw------- 1 root root 10, 183 Apr 28 17:42 /dev/hwrng
crw-rw-rw- 1 root root  1,   9 Apr 28 17:42 /dev/urandom
```

*列表 11-6：microSD 卡和 RNG 设备的权限*

你可以看到，对于设备文件，权限字符串指示它是块设备（`b`）还是字符设备（`c`）。结果还显示，microSD 卡只能被 root 或 *disk* 组的成员读取。对于 RNG，系统区分了操作系统提供的 RNG `urandom`，它可以被所有人读取和写入，以及硬件 RNG 设备 `hwrng`，只有 root 可以访问。

让我们将注意力从文件转向进程。列表 11-7 展示了典型的应用程序，如 Web 服务器 `httpd` 或 MQTT 经纪人 `mosquitto`，以及执行这些进程的相应用户。

```
# ps | grep -E 'PID|httpd|mosquitto'
  PID USER       VSZ STAT COMMAND
 1138 root      5680 S    /usr/sbin/httpd -DFOREGROUND -D SSL -D PHP5 -k start
 1148 daemon    224m S    /usr/sbin/httpd -DFOREGROUND -D SSL -D PHP5 -k start
 1149 daemon    224m S    /usr/sbin/httpd -DFOREGROUND -D SSL -D PHP5 -k start
 1150 daemon    224m S    /usr/sbin/httpd -DFOREGROUND -D SSL -D PHP5 -k start
 1235 mosquitt  6792 S    /usr/sbin/mosquitto -c /etc/mosquitto/mosquitto.conf
 2507 daemon    224m S    /usr/sbin/httpd -DFOREGROUND -D SSL -D PHP5 -k start
 2610 root      2320 S    grep -E PID|httpd|mosquitto
```

*列表 11-7：典型网络守护进程的用户上下文*

Web 服务器 `httpd` 展示了一种常见的策略，以限制在漏洞可能被远程利用时的攻击面。它以 root 用户启动，绑定到指定端口（例如，80），然后通过调用 `setgid()` 和 `setuid()` 分别改变其 GID 和 UID，从而故意降低其高权限。因此，在我的例子中，`httpd` 的四个“工作线程”在低权限用户 *daemon* 下运行。

对于 Mosquitto 经纪人也是如此。你可以从其文档中推断出，即使以 root 用户身份启动，Mosquitto 在读取其配置文件后立即降低权限，并继续以更有限的用户（在我的例子中是 `mosquitto`）的上下文运行。

`ps` 输出的用户名限制为八个字符。因此，`mosquitto` 显示为 `mosquitt`。

#### ***SSH 守护进程访问控制配置***

*Dropbear* 是一个轻量级的 SSH 守护进程，尤其在嵌入式系统中非常流行。它使设备能够进行安全的远程访问，这使它非常有用但也至关重要。像这样的应用程序在访问控制设置方面需要特别的关注，因为如果它们实施了“开放门政策”，那简直是在邀请攻击者。

列表 11-8 展示了从访问控制角度来看，`dropbear` 守护进程的一部分命令行参数。

```
# dropbear --help
...
-w  Disallow root logins
-G  Restrict logins to members of specified group
-s  Disable password logins
-g  Disable password logins for root
-B  Allow blank password logins
-T  Maximum authentication tries (default 10)
...
```

*列表 11-8：一些 dropbear 的访问控制选项*

禁用通过 SSH 的 root 访问（`-w`）在大多数情况下是个好主意。对于本案例研究，也可能合理限制 SSH 访问仅限于*service*组的用户（`-G`），因为这不应该是最终用户的功能。完全禁用密码登录并只允许公钥认证是完美的，但如果你的 PKI 还没有为此步骤做准备，那么`-s`选项就无法使用。由于我们已经完全禁用了 root 访问，使用`-g`就显得多余。`-B`参数只应在开发过程中使用；你可能不希望在生产固件镜像中看到这个。最后，你可以限制最大登录尝试次数——例如，限制为三次（`-T 3`）。

**注意**

*你还可以通过-p 选项将 dropbear SSH 守护进程的端口从 22 更改为自定义端口。这只是为了增加隐蔽性，但它可能会帮助你的联网设备免于被自动 SSH 扫描发现。*

要持久存储你的`dropbear`设置，你必须更改固件镜像中的*/etc/dropbear/default*文件。要实现上述访问控制限制，重要的一行内容是`DROPBEAR_EXTRA_ARGS="-w -T 3 -G service"`。

### **强制访问控制**

尽管 DAC 概念在嵌入式系统开发人员中通常是已知的，*强制访问控制（MAC）*往往是未知的领域。然而，MAC 实现可以显著提升设备的安全性，并在设备遭到破坏时限制损害。

嵌入式设备的 MAC 系统的基本思想是，关于用户和进程如何与文件和其他资源进行交互的权限和策略由制造商管理，并由操作系统强制执行。与 DAC 以用户为中心的方式不同，用户无法覆盖 MAC 定义的规则。

MAC 实现是强大的工具，但权力伴随着责任。*白名单*是一种流行的访问控制策略，默认情况下拒绝访问，只有显式允许时才授予访问权限。此方法也可以用于 MAC 系统，允许定义的主体对对象的访问。然而，如果你省略了指定某个合法访问为“允许”，也许是因为它很少被使用，那么在访问发生时，要求此访问的应用程序可能会在运行时崩溃。

如果你选择*黑名单*方法——只定义需要拒绝访问的危险情况，比如病毒扫描器检测到的恶意软件——则破坏功能的概率会降低。然而，你必须确保及时将新发现的恶意行为的相应规则添加到设备中。

#### ***Linux 安全模块***

由于 Linux 社区未能就一个特定的安全模块达成一致，Linux 引入了*Linux 安全模块（LSM）*框架。它使得在 Linux 上实现多种 MAC 系统成为可能。

这些 LSM 被编译到 Linux 内核中，并在内核代码中调用特定的钩子函数时采取相应的行动。这些钩子集成在操作系统中所有与访问控制相关的过程之中，从文件访问到任务生成，再到进程间通信。如果到达钩子，内核将控制权交给 LSM，LSM 至少可以记录执行的操作，或者根据其特定规则集直接决定是否允许访问。

不同的 LSM 实现方式在概念、配置规则集的方式以及支持社区方面有显著差异。然而，它们也有一个共同点：都对系统性能产生负面影响。接下来的章节将介绍一些流行的 LSM 实现。

#### ***SELinux***

2000 年，NSA 将其针对 Linux 的 MAC 系统理念发布给开源社区：*安全增强 Linux（SELinux）*。在该领域其他利益相关者的支持下，该项目蓬勃发展，并最终在 2003 年集成到主线 Linux 内核的 2.6 版本中。从版本 4.3 开始，它成为 Android 的默认 LSM，许多桌面和服务器应用的 Linux 发行版也都支持它。

SELinux 依赖于定义哪些对象可以被哪些主体访问的安全策略。为此，必须在 SELinux 中注册对象和主体，并为其分配相应的*标签*，标签中包含用户、角色和关联类型。这些标签定义了主体和对象的某种上下文或领域。实际的访问控制是通过*类型强制*实现的，类型强制定义了具有特定类型的主体是否可以访问具有特定类型的对象。

许多 Linux 发行版提供了自己预定义的 SELinux 策略集，以限制各种常见的应用和服务。此外，还有一个参考策略数据库，您可以根据需求使用。然而，为您的应用创建自定义策略需要深入了解其功能，并对 SELinux 的概念和结构有详细的理解。即便有工具可以支持您，也不应低估所需的特征化工作量。

在运行时，SELinux 可以以三种方式操作。*强制*模式适用于生产环境，因为它严格应用所有给定的策略并记录相应活动。然而，在开发或测试阶段，*宽容*模式更为适用。它处理所有策略，但仅生成警告和日志数据，而不强制执行定义的规则。这对于微调自定义策略和故障排除非常有帮助。如果*禁用*，SELinux 将完全关闭，不会保护或限制任何内容。

尽管（或者也许正因为）SELinux 提供了大量的能力，它仍然是一个相当复杂的工具，许多嵌入式系统工程师因此不愿使用它。这可能是其他 LSM 实现出现并成为流行替代方案的主要原因，如接下来的部分所述。

#### ***AppArmor***

*AppArmor*是第二个在 Linux 发行版中获得显著普及的 LSM 实现。它于 2010 年成为 Linux 内核 2.6.36 的一部分，目前是 Ubuntu 和 SUSE Linux 的默认 MAC 系统。自 2009 年以来，它的开发得到了 Canonical 的资助。

访问控制是基于每个应用程序的个人配置文件进行管理的。与 SELinux 不同，AppArmor 使用文件系统路径来识别主体和文件对象，因此其语法具有更好的可读性。此外，它还支持混合的白名单和黑名单规则方法，控制进程的资源访问。创建的配置文件可以限制网络访问和各种 Linux 能力，还可以限制读取、写入和执行文件的权限。

AppArmor 提供了一些预定义的配置文件，Ubuntu 社区还维护了常见应用程序的额外配置文件。此外，AppArmor 还提供了多个工具，帮助开发者为自定义应用程序配置文件并生成相应的配置文件。

一般来说，有两种方式来描述访问需求。首先，目标化的配置文件方法允许你捕获单个应用程序的访问事件，并从中自动生成配置文件。其次，AppArmor 可以应用系统监控方法，记录一组定义应用程序的访问操作，持续数天甚至数周，并跨多个重启。收集的日志信息可以转化为一系列配置文件，以尽可能优化的方式限制分析过的应用程序。

设备上的每个 AppArmor 配置文件在运行时可以处于三种模式之一：强制、抱怨或审计。在*强制模式*下，配置文件设定的规则会被强制执行，任何违规尝试都会被记录。*抱怨模式*允许监控在定义的配置文件下应用程序的行为，违规操作会被记录。这个模式也用于前面提到的自动化配置文件创建，因此有时被称为 AppArmor 的*学习模式*。为了在强制执行给定策略的同时记录所有访问，无论是否成功，必须选择*审计模式*。

AppArmor 是一个值得推荐的 SELinux 替代方案，成功地降低了配置和配置文件的复杂性。从安全性角度来看，它有时比竞争对手更加宽松，某些情况下可能存在绕过访问控制的空间。然而，对于嵌入式系统来说，它可能是引入 MAC 机制的完美折衷方案。

#### ***其他 LSM 和非 LSM MACs***

除了两个流行的 LSM 实现，SELinux 和 AppArmor，你还可以考虑其他选项。

正如其名称所示，*简化强制访问控制内核（SMACK）*系统的开发重点是简化，与 SELinux 的复杂性相对。自 2008 年起，它一直是 Linux 主线内核的一部分，并始终旨在用于嵌入式系统。两个较大的操作系统项目依赖于其保护机制：用于三星智能电视的移动操作系统 Tizen，以及旨在为联网汽车提供开源平台的汽车级 Linux 发行版。然而，查看 SMACK 的官方网站及其 Git 仓库，似乎它不再积极维护。

基于 LSM 框架的另一个 MAC 系统被称为*TOMOYO*。该项目始于 2003 年，并在 2009 年合并进 Linux 内核 2.6.30。其动机是为了简化使用和提高可用性——例如，通过自动生成策略来实现，这也是必要的，因为这个 MAC 系统并没有为常见服务提供一套全面的规则。此外，TOMOYO 不仅作为 MAC 实现，还促进了系统行为分析。它有三个版本：1.*x*、2.*x*和 AKARI。第一个版本需要特定的内核补丁，因此通常不是首选。AKARI 和 TOMOYO 2.*x*使用 LSM 框架。在撰写本文时，AKARI 提供了一些额外的功能，但 TOMOYO 2.*x*正在赶超。

虽然 LSM 框架为集成自定义安全模块提供了多种可能性，但并非所有社区成员都对其实现感到满意，尤其是考虑到它带来的性能开销。因此，也存在非 LSM 的 MAC 系统，旨在提高性能或增强安全模块功能。然而，由于这些实现不是主线内核的一部分，必须通过应用自定义补丁集来集成，它们可能只有在你无法通过流行的 LSM 实现满足需求时才是一个可行的选择。

### **案例研究：使用 AppArmor 进行应用程序限制**

在本案例研究中，我将重点介绍如何在我的 STM32MP157F-DK2 设备上使用 Yocto 工具链安装 AppArmor，并演示如何通过基本使用来限制应用程序。

#### ***安装***

AppArmor 不包含在 ST 的 OpenSTLinux 发行版的默认安装中。幸运的是，Armin Kuster 维护的 Yocto `meta-security`层提供了一个位于*meta-security/recipes-mac/AppArmor*下的 AppArmor 食谱。

克隆相应的 Git 仓库后，可以通过清单 11-9 中所示的设置，配置 Linux 内核以使用 AppArmor。

```
CONFIG_SECURITY=y
CONFIG_SECURITY_APPARMOR=y
CONFIG_DEFAULT_SECURITY="apparmor"
CONFIG_SECURITY_APPARMOR_BOOTPARAM_VALUE=1
```

*清单 11-9：启用 AppArmor 的 Linux 内核配置*

前两行启用 AppArmor，最后两行则将其设置为默认使用的 LSM。然而，我还需要在 U-Boot 的*extlinux.conf*文件中添加`security=apparmor`，以在启动时选择 AppArmor。

要编译和安装 AppArmor 用户空间工具，请在镜像的配方中添加 `IMAGE_INSTALL += "apparmor"` 这一行。我还必须向提供的 OpenSTLinux 添加几个发行版特性，如 列表 11-10 所示，以便使 Yocto 成功完成构建过程。

```
DISTRO_FEATURES += "security"
DISTRO_FEATURES += "apparmor"
DISTRO_FEATURES += "tpm"
```

*列表 11-10：来自* meta-security *的 AppArmor 发行版特性*

启动设备后，您可以使用 列表 11-11 中显示的命令检查 AppArmor 是否已正确启用。如果返回 `Y`，则表示已正确激活。

```
# cat /sys/module/apparmor/parameters/enabled
Y
```

*列表 11-11：检查是否正确启用了 AppArmor*

AppArmor 附带了 `aa-status` 工具，它列出了有关 AppArmor 当前状态的各种详细信息，如 列表 11-12 所示。

```
# aa-status
apparmor module is loaded.
50 profiles are loaded.
50 profiles are in enforce mode.
   ...
   apache2
   ...
   avahi-daemon
   ...
 ping
   ...
   syslogd
   traceroute
   ...
0 profiles are in complain mode.
...
2 processes are in enforce mode.
   /usr/sbin/avahi-daemon (665) avahi-daemon
   /usr/sbin/avahi-daemon (667) avahi-daemon
0 processes are in complain mode.
...
```

*列表 11-12：初始的* aa-status *输出*

您可以看到来自 `meta-security` 层的 AppArmor 配方也安装了一组 50 个标准配置文件，这些配置文件在我的设备上以强制模式加载。然而，我首先注意到的是，尽管为 `apache2` 和 `syslogd` 加载了配置文件，但相应的当前运行的进程并未受到限制。只有 `avahi-daemon` 进程根据其配置文件受到限制。

要调查此问题，我们需要查看存储在 */etc/apparmor.d/* 中的默认 AppArmor 配置文件。对于 `apache2`，包含提供的配置文件的文件名为 *usr.sbin.apache2*。文件名已经暗示了它所限制的可执行文件路径：*/usr/sbin/apache2*。查看文件内容，可以看到一行 `profile apache2 /usr/\{bin, sbin\}/apache2`，这意味着当前的配置文件名为 `apache2`，并且目标是位于 */usr/bin/* 或 */usr/sbin/* 的可执行文件 *apache2*。

不幸的是，在我的安装中没有这个文件。相反，它被命名为 *httpd*。因此，我创建了一个名为 *usr.sbin.httpd* 的初始文件副本。我还将配置文件名称更改为 `httpd`，并将可执行文件的路径更改为 `/usr/\{bin,sbin\}/httpd`。然后，我按 列表 11-13 中所示的方式加载了该配置文件，并重新启动了 Web 服务器。

```
# aa-enforce /etc/apparmor.d/usr.sbin.httpd
Setting /etc/apparmor.d/usr.sbin.httpd to enforce mode.
# systemctl restart apache2
# aa-disable /etc/apparmor.d/usr.sbin.apache2
Disabling /etc/apparmor.d/usr.sbin.apache2.
```

*列表 11-13：加载和禁用配置文件*

我还通过 `aa-disable` 禁用了原始配置文件，以便清理。

列表 11-14 展示了另一次调用 `aa-status` 的输出，显示 `httpd` 配置文件已正确加载，并且所有四个相应的进程实例都按预期在强制模式下运行。

```
# aa-status
...
50 profiles are loaded.
50 profiles are in enforce mode.
...
   httpd
   httpd//DEFAULT_URI
   httpd//HANDLING_UNTRUSTED_INPUT
   httpd//phpsysinfo
...
6 processes are in enforce mode.
   /usr/sbin/avahi-daemon (669) avahi-daemon
   /usr/sbin/avahi-daemon (672) avahi-daemon
   /usr/sbin/httpd (668) httpd
   /usr/sbin/httpd (678) httpd
   /usr/sbin/httpd (679) httpd
   /usr/sbin/httpd (681) httpd
...
```

*列表 11-14：更改配置文件后的* aa-status *输出*

尽管我们成功激活了预定义的配置文件，但我们不知道该配置文件是否以安全的方式实际限制了 Web 服务器应用程序。给定配置文件中的一条注释说：“这个配置文件完全是宽松的”，这意味着您仍然需要根据您的应用程序和需求对其进行定制。

快速查看与`syslogd`工具关联的 *sbin.syslogd* 配置文件，这个工具在本小节开头被标识为第二个示例二进制文件，可以发现配置的路径 */sbin/syslogd* 与相应可执行文件的路径一致，但进程仍然没有以强制模式运行。如清单 11-15 所示，二进制文件的属性显示，该可执行文件实际上是指向另一个可执行文件的符号链接——即 */bin/busybox.nosuid*。

```
# ls -l /sbin/syslogd
lrwxrwxrwx 1 root root 19 ... /sbin/syslogd -> /bin/busybox.nosuid
```

*清单 11-15：指向另一个可执行文件的符号链接*

这种情况有些复杂，因为 BusyBox 将多种工具合并在一个二进制文件中。仅仅更改 `syslogd` 配置文件的路径并不能解决这个问题，反而会导致其他 BusyBox 功能出现问题。在这种情况下，你有几个选择。你可以只放宽 `syslogd` 配置文件，也可以搜索或创建一个全面的 `busybox` 配置文件，或者你也可以最终安装并使用原版 `syslogd` 应用程序。

#### ***应用程序剖析***

对于你自己的应用程序或没有预定义 AppArmor 配置文件的第三方工具，如果你希望在运行时通过 MAC 机制限制它们，你必须自己创建一个配置文件。

我们来看一个被简化到最小的 Python 应用。清单 11-16 展示了代码。

```
#!/usr/bin/python3

import sys

if len(sys.argv) == 2:
    file_path = sys.argv[1]
    with open(file_path, 'r') as f:
        print(f.read())
else:
    print('Usage:', sys.argv[0], '<filename>')
```

*清单 11-16：一个简单的 Python 打印文件应用*

这个应用程序的唯一目的是打印作为命令行参数给定的文本文件的内容。应用程序的文件名是 *printfile.py*，位于 */home/root/* 目录下，并且被标记为可执行。

假设这个工具是你 web 界面的重要组成部分，并且需要超级用户权限运行，因为它必须打印 *testfile* 和 *logfile* 文件的内容，这些文件只能由 root 访问。然而，在进行威胁和风险分析时，你发现攻击者可能能够注入除了两个预定文件路径之外的其他路径，这可能会导致敏感信息泄露，必须加以防范——例如，通过使用量身定制的 AppArmor 配置文件。

清单 11-17 展示了我在 */etc/apparmor.d/home.root.printfile.py* 中创建的基本初始配置文件，作为配置此应用程序的起点。它包括对之前提到的两个文件的读取权限（`r`），并拒绝任何其他文件访问。

```
/home/root/printfile.py {
   /home/root/testfile     r,
   /home/root/logfile      r,
}
```

*清单 11-17：*printfile.py 的初始 AppArmor 配置文件

在第二步中，我以告警模式加载了新创建的配置文件，如清单 11-18 所示。

```
# aa-complain /etc/apparmor.d/home.root.printfile.py
Setting /etc/apparmor.d/home.root.printfile.py to complain mode.
```

*清单 11-18：以告警模式加载配置文件*

如果你现在在 *home/root/* 中执行 `./printfile.py testfile`，应用程序将正常运行，但会为所有配置文件违规情况创建日志条目。

清单 11-19 展示了与 *printfile.py* 相关的精简版 AppArmor 内核消息。

```
# dmesg | grep printfile.py
... audit: type=1400 audit(1652997509.490:132): apparmor="STATUS"
             operation="profile_load" profile="unconfined"
             name="/home/root/printfile.py" pid=1557 comm="apparmor_parser"
... audit: type=1400 audit(1652997771.210:133): apparmor="ALLOWED"
             operation="open" profile="/home/root/printfile.py"
             name="/etc/ld.so.cache" pid=1560 comm="printfile.py"
             requested_mask="r" denied_mask="r" fsuid=0 ouid=0
... audit: type=1300 audit(1652997771.210:133): arch=40000028 ...
             comm="printfile.py" exe="/usr/bin/python3.10"
             subj=/home/root/printfile.py (complain) key=(null)
... audit: type=1400 audit(1652997771.210:134): apparmor="ALLOWED"
             operation="open" profile="/home/root/printfile.py"
             name="/usr/lib/libpython3.10.so.1.0" pid=1560 comm="printfile.py"
             requested_mask="r" denied_mask="r" fsuid=0 ouid=0
... audit: type=1300 audit(1652997771.210:134): arch=40000028 ...
             comm="printfile.py" exe="/usr/bin/python3.10"
             subj=/home/root/printfile.py (complain) key=(null)
... audit: type=1400 audit(1652997771.210:135): apparmor="ALLOWED"
             operation="file_mmap" profile="/home/root/printfile.py"
             name="/usr/lib/libpython3.10.so.1.0" pid=1560 comm="printfile.py"
             requested_mask="rm" denied_mask="rm" fsuid=0 ouid=0
... audit: type=1300 audit(1652997771.210:135): arch=40000028 ...
             comm="printfile.py" exe="/usr/bin/python3.10"
             subj=/home/root/printfile.py (complain) key=(null)
... audit: type=1400 audit(1652997771.210:136): apparmor="ALLOWED"
             operation="open" profile="/home/root/printfile.py"
             name="/lib/libc.so.6" pid=1560 comm="printfile.py"
             requested_mask="r" denied_mask="r" fsuid=0 ouid=0
```

*Listing 11-19：* printfile.py *的 AppArmor 抱怨消息*

你可以看到，在以预期的方式调用*printfile.py*时发生了多个访问违规。如果你将初始配置文件放入强制模式，应用程序将不再工作。因此，你需要使用所示输出扩展*printfile.py*的 AppArmor 配置文件。例如，你需要授予*/etc/ld.so.cache*的读取权限（`r`），*/usr/bin/python3.10*的执行权限（`ux`），以及读取（`r`）和映射（`m`）*/usr/lib/libpython3.10.so.1.0*的权限。

Listing 11-20 展示了在抱怨模式下执行四次、配置文件优化和重新加载后的最终配置文件。

```
/home/root/printfile.py flags=(complain) {
   /home/root/testfile             r,
   /home/root/logfile              r,
   /etc/ld.so.cache                r,
   /usr/bin/python3.10             ux,
   /usr/lib/libpython3.10.so.1.0   rm,
   /lib/libc.so.6                  rm,
   /lib/libm.so.6                  rm,
   /usr/lib/locale/locale-archive  r,
   /usr/lib/python3.10/            r,
   /usr/lib/python3.10/**          r,
   /home/root/printfile.py         r,
}
```

*Listing 11-20：优化后的* printfile.py *的 AppArmor 配置文件*

在这个手动表征阶段之后，创建的配置文件可以在强制模式下加载，并测试其行为（Listing 11-21）。

```
# aa-enforce /etc/apparmor.d/home.root.printfile.py
Setting /etc/apparmor.d/home.root.printfile.py to enforce mode.
# ./printfile.py testfile
--- --- ---
This is a test file!
--- --- ---
# ./printfile.py logfile
--- --- ---
All the logs...
--- --- ---
# ./printfile.py secrets
Traceback (most recent call last):
  File "/home/root/./printfile.py", line 7, in <module>
    with open(file_path, 'r') as f:
PermissionError: [Errno 13] Permission denied: 'secrets'
# ./printfile.py /etc/passwd
Traceback (most recent call last):
  File "/home/root/./printfile.py", line 7, in <module>
    with open(file_path, 'r') as f:
PermissionError: [Errno 13] Permission denied: '/etc/passwd'
```

*Listing 11-21：在强制模式下测试* printfile.py *

打印*testfile*和*logfile*文件按预期工作。然而，如果攻击者尝试读取同一文件夹中的*secrets*文件，甚至是*/etc/passwd*文件，AppArmor 将成功防止严重损害。

这个简单的案例研究展示了应用程序表征及相应的 AppArmor 配置文件创建的基本可行性。然而，即使是这个简单的示例也经历了多个配置文件迭代，且生成的配置文件将需要持续维护——例如，如果你切换到另一个发行版或甚至更新版本的 Python。此外，正如你可以想象的，复杂的应用程序需要显著更多的表征和测试工作。

### **总结**

访问控制是一个极其广泛的话题，可以单独写成一本书。它涵盖了用户、组、目录结构和访问权限的基本配置（本章讨论了 Linux DAC 系统），禁用不适用于终端用户的硬件调试功能和工具，以及必须根据特定应用程序的行为和资源访问需求进行微调的操作系统强制 MAC 策略的复杂领域。尽管我不指望你把所有时间都用来设计完美的访问控制设置，但在安全设备工程中，绕过这个问题是不可行的。

实际操作中，我们必须找到切实可行的折中方案。完全的白名单策略可能难以实施，并且如果配置错误，可能会导致应用程序崩溃，从而受到安全指责。另一方面，黑名单策略则无法立即捕捉到出现的新威胁。权限的精细粒度只能通过时间管理，如果你愿意付出大量努力，但如果你的访问控制概念过于简单，敌人将会感谢你移除了那个恼人的障碍。

最后，我想指出，访问控制机制总是与系统完整性保护有着密切关系，如第八章所述。试想一下，如果你辛苦定义了一套完美的访问规则，却发现攻击者可以在几分钟内将它们全部重置为 777，那将非常痛苦。
