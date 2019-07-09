-------------
词汇表
-------------

Nutanix Core
++++++++++++

AOS
...

AOS代表Acropolis操作系统，它是在Controller VMs（CVM）上运行的操作系统。

Pulse 脉冲
.....

Pulse为Nutanix客户支持团队提供诊断系统数据，以便他们能够为Nutanix解决方案提供主动的上下文感知支持。

Prism Elemen
.............

Prism Element是Nutanix的原生管理平台。由于其设计基于消费者产品界面，因此它比许多企业应用程序界面更直观，更易于使用。

Prism Central
.............

Prism Central是Nutanix的多云控制和管理界面。 Prism Central可以管理多个Nutanix群集，并作为监控和分析的聚合点。

Node
....

行业标准x86服务器，带有服务器直接的SSD和可选的HDD（全闪存和混合选项）。

Block
.....

2U机架式机箱，包含1,2或4个节点，共享电源和风扇，没有共享的背板。

Storage Pool
............

存储池是一组物理存储设备，包括用于群集的PCIe SSD，SSD和HDD设备。

Storage Container
.................

Storage Container 容器是用于实现存储策略的可用存储的子集。

读I/O的解析
.....................

性能和可用性

 - 数据在本地读取
 - 仅当数据不在本地时才进行远程访问

写入I/O的解析
......................

性能和可用性

 - 数据是在本地写入的
 - 在其他节点上复制以获得高可用性
 - 副本分布在群集中以获得高性能

Nutanix Flow
++++++++++++

Application Security Policy
...........................

如果要通过指定允许的流量源和目标来保护应用程序，请使用Application Security Policy"应用程序安全策略"。

Isolation Environment Policy
............................

如果要阻止由其类别标识的两组VM之间的所有流量（无论方向如何），请使用Isolation Environment Policy"隔离环境策略"。组内的VM可以相互通信。

Quarantine Policy
.................

如果要隔离受感染或受感染的VM并且可选择要对其进行取证，请使用Quarantine Policy"隔离策略"。您无法修改此政策。隔离VM的两种模式是Strict或Forensic。

Strict 严格：如果要阻止所有入站和出站流量，请使用此设置。

Forensic 取证：如果要阻止除包含取证工具的类别的流量之外的所有入站和出站流量，请使用此设置。

AppTier
.......

将应用程序中的层（例如web，application_logic和database）的值添加到此类别，并在配置安全策略时使用这些值将应用程序划分为层。

AppType
.......

将应用程序中的VM与适当的内置应用程序类型（如Exchange和Apache_Spark）相关联。您还可以更新类别，以便为此类别中未列出的应用程序添加值。

Environment
...........

为要彼此隔离的环境添加值，然后将VM与值关联。
