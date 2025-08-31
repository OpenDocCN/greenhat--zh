

# 第九章：B SDDL SID 别名映射



![](img/chapter.jpg)

第五章介绍了安全描述符定义语言（SDDL）格式，用于将安全描述符表示为字符串，并提供了一些 Windows 支持的、用于表示知名 SDDL SID 的两字符别名示例。尽管微软文档中有关于 SID 的 SDDL 格式，但并未提供单一资源列出所有短 SID 别名字符串。唯一可用的资源是 Windows SDK 中的 *sddl.h* 头文件。该头文件定义了程序员可以用来操作 SDDL 格式字符串的 Windows API，并提供了短 SID 别名字符串的列表。

表 B-1 包含了简短的别名以及它们所代表的名称和完整的 SID。该表格摘自 Windows 11 SDK 提供的头文件（操作系统版本为 22621），应当是撰写时的权威列表。请注意，某些 SID 别名仅在您连接到域网络时有效。您可以通过 SID 名称中的 *<DOMAIN>* 占位符来识别这些别名，您需要将其替换为系统连接的域名。同时，将 SDDL SID 字符串中的 <DOMAIN> 占位符替换为唯一的域 SID。

表 B-1： 支持的 SDDL SID 别名到 SID 的映射

| SID 别名 | 名称 | SDDL SID |
| --- | --- | --- |
| AA | 内建\访问控制协助操作员 | S-1-5-32-579 |
| AC | 应用程序包权限\所有应用程序包 | S-1-15-2-1 |
| AN | NT 权限\匿名登录 | S-1-5-7 |
| AO | 内建\帐户操作员 | S-1-5-32-548 |
| AP | <DOMAIN>\受保护用户 | S-1-5-21-<DOMAIN>-525 |
| AS | 认证机构声明身份 | S-1-18-1 |
| AU | NT AUTHORITY\已认证用户 | S-1-5-11 |
| BA | 内置\管理员 | S-1-5-32-544 |
| BG | 内置\访客 | S-1-5-32-546 |
| BO | 内置\备份操作员 | S-1-5-32-551 |
| BU | 内置\用户 | S-1-5-32-545 |
| CA | <DOMAIN>\证书发布者 | S-1-5-21-<DOMAIN>-517 |
| CD | 内置\证书服务 DCOM 访问 | S-1-5-32-574 |
| CG | 创建者组 | S-1-3-1 |
| CN | <DOMAIN>\可克隆域控制器 | S-1-5-21-<DOMAIN>-522 |
| CO | 创建者所有者 | S-1-3-0 |
| CY | 内置\加密操作符 | S-1-5-32-569 |
| DA | <DOMAIN>\域管理员 | S-1-5-21-<DOMAIN>-512 |
| DC | <DOMAIN>\域计算机 | S-1-5-21-<DOMAIN>-515 |
| DD | <DOMAIN>\域控制器 | S-1-5-21-<DOMAIN>-516 |
| DG | <DOMAIN>\域访客 | S-1-5-21-<DOMAIN>-514 |
| DU | <DOMAIN>\域用户 | S-1-5-21-<DOMAIN>-513 |
| EA | <DOMAIN>\企业管理员 | S-1-5-21-<DOMAIN>-519 |
| ED | NT AUTHORITY\企业域控制器 | S-1-5-9 |
| EK | <DOMAIN>\企业密钥管理员 | S-1-5-21-<DOMAIN>-527 |
| ER | BUILTIN\事件日志读取器 | S-1-5-32-573 |
| ES | BUILTIN\RDS 端点服务器 | S-1-5-32-576 |
| HA | BUILTIN\Hyper-V 管理员 | S-1-5-32-578 |
| HI | 强制标签\高级强制级别 | S-1-16-12288 |
| IS | BUILTIN\IIS_IUSRS | S-1-5-32-568 |
| IU | NT AUTHORITY\交互式 | S-1-5-4 |
| KA | <DOMAIN>\关键管理员 | S-1-5-21-<DOMAIN>-526 |
| LA | <DOMAIN>\管理员 | S-1-5-21-<DOMAIN>-500 |
| LG | <DOMAIN>\访客 | S-1-5-21-<DOMAIN>-501 |
| LS | NT AUTHORITY\本地服务 | S-1-5-19 |
| LU | BUILTIN\性能日志用户 | S-1-5-32-559 |
| LW | 强制标签\低强制级别 | S-1-16-4096 |
| ME | 强制标签\中等强制级别 | S-1-16-8192 |
| MP | 强制标签\中级强制级别 | S-1-16-8448 |
| MS | BUILTIN\RDS 管理服务器 | S-1-5-32-577 |
| MU | BUILTIN\性能监视器用户 | S-1-5-32-558 |
| NO | BUILTIN\网络配置操作员 | S-1-5-32-556 |
| NS | NT AUTHORITY\NETWORK SERVICE | S-1-5-20 |
| NU | NT AUTHORITY\NETWORK | S-1-5-2 |
| OW | 所有者权限 | S-1-3-4 |
| PA | <DOMAIN>\组策略创建者所有者 | S-1-5-21-<DOMAIN>-520 |
| PO | BUILTIN\打印操作员 | S-1-5-32-550 |
| PS | NT AUTHORITY\SELF | S-1-5-10 |
| PU | BUILTIN\高级用户 | S-1-5-32-547 |
| RA | BUILTIN\RDS 远程访问服务器 | S-1-5-32-575 |
| RC | NT AUTHORITY\RESTRICTED | S-1-5-12 |
| RD | BUILTIN\远程桌面用户 | S-1-5-32-555 |
| RE | BUILTIN\复制器 | S-1-5-32-552 |
| RM | BUILTIN\远程管理用户 | S-1-5-32-580 |
| RO | <DOMAIN>\只读域控制器 | S-1-5-21-<DOMAIN>-498 |
| RS | <DOMAIN>\RAS 和 IAS 服务器 | S-1-5-21-<DOMAIN>-553 |
| RU | BUILTIN\预 Windows 2000 兼容访问 | S-1-5-32-554 |
| SA | <DOMAIN>\架构管理员 | S-1-5-21-<DOMAIN>-518 |
| SI | 强制标签\系统强制级别 | S-1-16-16384 |
| SO | BUILTIN\服务器操作员 | S-1-5-32-549 |
| SS | 服务声明身份 | S-1-18-2 |
| SU | NT AUTHORITY\服务 | S-1-5-6 |
| SY | NT AUTHORITY\系统 | S-1-5-18 |
| UD | NT AUTHORITY\用户模式驱动程序 | S-1-5-84-0-0-0-0-0 |
| WD | 每个人 | S-1-1-0 |
| WR | NT AUTHORITY\写入限制 | S-1-5-33 |
