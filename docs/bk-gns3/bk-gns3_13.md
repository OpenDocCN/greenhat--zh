# 附录 C. NM-16ESW 和 IOU L2 限制

GNS3 Dynamips NM-16ESW 模块与其真实的 Cisco 对应模块操作相同。与物理模块一样，Dynamips 模块不支持所有 Cisco Catalyst 交换机的功能，意味着获得完整 Catalyst 交换机功能的唯一保证方法是将一个或多个真实的交换机集成到 GNS3 项目中。我在本附录中列出了那些不受支持的功能，以及 IOU L2 镜像可能不支持的功能。

如果你想使用某个特定的 Catalyst 交换机功能，首先检查这些列表，看看是否需要使用物理交换机。

# 不支持的 NM-16ESW 特性

NM-16ESW 交换机模块不支持以下功能：

+   接入交换设备管理器（SDM）模板

+   ACL – 改进的合并算法

+   ARP 优化

+   BGP 增强对编号路径访问控制列表的支持，最多 500 条

+   BGP 在最大前缀限制到达后重启邻居会话

+   BGP 路由映射继续支持外发策略

+   清除每个端口的计数器

+   DHCP 嗅探

+   DHCP 嗅探计数器

+   启动时的诊断选项

+   错误禁用端口重新激活

+   错误禁用超时

+   EtherChannel

+   EtherChannel – 灵活的 PAgP

+   EtherChannel 保护

+   回退桥接

+   Flex 链路双向快速收敛

+   Flex Link VLAN 负载均衡

+   Flex 链路接口抢占

+   GOLD – 通用在线诊断

+   IEEE 802.1ab 链路层发现协议

+   IEEE 802.1s – 多重生成树（MST）标准兼容

+   IEEE 802.1s VLAN 多重生成树

+   IEEE 802.1t

+   IEEE 802.1W 生成树快速重新配置

+   IEEE 802.1x 认证失败时开放

+   IEEE 802.1x 认证失败 VLAN

+   IEEE 802.1x VLAN 分配

+   IEEE 802.1x 唤醒 LAN 支持

+   IEEE 802.1x 认证器

+   IEEE 802.1x 多域认证

+   IEEE 802.1x 与端口安全

+   IEEE 802.1x RADIUS 计费

+   IEEE 802.3ad 链路聚合（LACP）

+   IEEE 802.3af 以太网供电

+   IGMP 快速离开

+   IGMP 第 1 版

+   IGRP

+   IP 电话检测增强

+   IP 电话增强 – 物理环路检测

+   IPSG（IP 源保护）

+   巨型帧

+   L2PT – 第二层协议隧道

+   MAC 认证绕过

+   MLD 嗅探

+   多播 EtherChannel 负载均衡

+   NAC – L2 IEEE 802.1x

+   NAC – L2 IP

+   NAC – L2 IP 与认证失败时开放

+   基于数据包的风暴控制

+   每端口每 VLAN 流量控制

+   端口安全

+   私有 VLAN 端口的端口安全

+   私有 VLAN

+   QoS 策略通过边界网关协议（QPPB）传播

+   快速每 VLAN 生成树（Rapid-PVST）

+   减少的 MAC 地址使用

+   远程 SPAN（RSPAN）

+   智能端口

+   生成树协议（STP） – 环路保护

+   生成树协议（STP） – 端口快速 BPDU 过滤

+   生成树协议（STP） – 端口快速支持中继链路

+   生成树协议（STP） – 根保护

+   生成树协议（STP） – 上行链路负载均衡

+   SRR（有形轮询）

+   热备份主管端口使用

+   STP 系统日志消息

+   交换数据库管理器（SDM）

+   中继链路故障转移

+   可信边界（为 CDP 设备扩展信任）

+   单播 MAC 过滤

+   单向链路检测（UDLD）

+   VLAN 访问控制列表（VACL）

+   VLAN 感知端口安全

+   加权尾丢弃（WTD）

# Cisco IOU L2 镜像中的不支持功能

据说 IOU 支持大约 90%的实际 Cisco Catalyst 交换机功能。以下功能可能不被 IOU L2 镜像支持。然而，由于 Cisco 负责更新 IOU 镜像，实际情况可能有所不同。

+   DHCP 嗅探

+   ISL 中继

+   L3 Etherchannel

+   MLS QoS

+   端口安全

+   私有 VLAN

+   QinQ

+   SPAN/RSPAN/ERSPAN

+   UDLD

+   语音 VLAN
