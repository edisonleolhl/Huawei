#二、OpenStack
> 本文包括：

> 1、OpenStack 起源

> 2、OpenStack 架构

> 3、Nova 介绍

> 4、Swift 介绍

> 5、Keystone 介绍

> 6、Neutron 介绍

> 7、Glance 介绍

> 8、Cinder 介绍

> 9、Ceilometer 介绍

> 10、Heat 介绍



- OpenStack 不是云
- OpenStack 不是虚拟化
- OpenStack 是目前最主流的开源云操作系统内核
- 三大组件
	- 计算
	- 网络
	- 存储
- 主要的用户接口
	- Dashboard
	- Horizon
##1、OpenStack 起源
- 起源：2010.7
 
- rack space （美国公司）开源了存储代码  Swift
 
- NASA 贡献了计算代码 Nova
 
- OpenStack 不是一个项目，而是一些项目的统称

- OpenStack 项目发展状况：
	- **提供对象存储服务的 Swift**
	- **提供计算服务的 Nova**
	- 提供镜像服务的 Glance
	- 提供网络服务的 Neutron
	- 提供身份认证服务的 Keystone
	- 提供计量服务的 Ceiliometer
	- 提供块存储服务 Cinder
	- 提供编排服务的 Heat

- OpenStack 的会员构成：
	- 白金会员【8 个】
	- 黄金会员【华为。。。】
	- 其他的企业赞助商

- OpenStack 的版本：
	- 按照 26 个字母顺序来进行区分的
 
- 目前是 Ocata 2017.2.22
	- 视频录制时说的是：Newton 2016.10.06(视频中)

##2、OpenStack 架构
1. 提供身份认证服务的 Keystone 组件
	- 主要负责身份服务
	- 管理用户、租户、角色、服务和服务断点
	- 可以支持 SQL，PAM，LDAP 作为后端
2. 提供计算服务的 Nova 组件
	- 主要负责虚拟机实例的调度分配以及实例的创建、起停、迁移、重启等操作，从而来管理云中实例的生命周期。
	- 是整个云中的组织控制器
	- 计算服务：
		- 计算节点运行虚拟机的
	- 分布式控制器
		- 负责处理器调度策略及 API 调用等。 
3. 提供镜像服务的 Glance 组件
	- 能够实现镜像的创建、镜像快照管理、以及镜像模板等等。
	- 同时也支持各种格式的镜像格式，如：raw，qcow，vhd，vmdk，iso
	- 镜像文件一般存储在后端，如：Swift，Filesystem，AmazonS3
4. 提供对象存储服务的 Swift
	- 主要提供了存取数据的应用服务。
	- 通常用于保存非结构化的数据，比如通常作为 Glance 组件的存储后端或者作为一些云盘等应用
5. 提供网络服务的 Neutron
	- 基于软件定义网络的思想，实现网路资源的软件化管理。支持各种各样类型的插件，实现多租户网络的隔离。
	- 也可以对硬件以及软件的解决方案进行集成。
6. 提供块存储服务 Cinder
	- 为虚拟机实例提供卷的持久化服务（块存储）
	- 同时支持一些对卷的快照、备份等的一些管理
	- 基于插件的架构，非常易于扩展。
7. 提供 WEB 统一化管理界面服务的 HORIZON 组件
	- 主要提供了自动化仪表板的管理服务
	- 实现对用户、租户、卷、网络等几乎多有资源的图形化管理。
8. 提供监控和计量服务的 Ceilometer
	- 提供对 Opens tack 中平台组件的监控
	- 提供计量服务

##3、Nova 介绍
- 什么是 Nova?
	- Nova 是 Open stack 云中的计算组织控制器，管理 opens tack 云中实例的生命周期的所以活动，使得 Nova 称为一个负责管理计算资源、网络、认证所需的可扩展性平台。
2. 常用术语
	- KVM：内核虚拟化，Open Stack 默认的 Hypervisor层
	- Qemu: KVM 的替补角色，没有 KVM 的执行效率高，且不支持全虚拟化
	- Flavor：新建虚拟机的配置列表，虚拟机模板
	- Keypair：ssh 连接访问实例的密钥对
	- 安全组：是一个控制访问策略的容器，里面包含了各种各样的安全组规则
	- 安全组规则：用来控制实例访问的具体策略
- Nova 组件
	- Nova API：提供了统一风格的 RestAPI 接口，作为 Nova 组件的入口，接受用户的请求
	- Nova scheduler ：负责调度，将实例分配到具体的计算节点
	- Nova conductor：主要负责与 Nova 数据库进行交互
	- Nova compute：运行在计算节点商，用于虚拟机实例的创建和管理
	以及提供消息传递的
	- 消息队列：Nova 各个组件之间的消息传递
和数据库模块：

- **【Nova 各个组件如何协作运行？】**
	1.  首先，用户通过 CLI 命令行或者 horizon 向 Nova 组件提出创建实例的请求时，Nova API 作为 Nova 的入口，将会接受用户的请求，将会以消息队列的方式，将请求发送给 Nova scheduler
	- Nova scheduler 从消息队列中，侦听到 Nova API 的消息队列后，去数据库中查询，当前计算节点的负载和使用情况
	- 由于 Nova scheduler 不能直接跟数据库进行交互，因此，将会借助于消息队列的方式通过 Nova conductor 组件，进而与数据库进行交互
	- 然后将查询到的结果，将虚拟机实例分配到当前负载最小，并且满足启动虚拟机实例的那个计算节点上
	- 但最终的虚拟机实例的组件，还是要靠 Nova compute 来完成
	- 实例的创建，离不开镜像、网络等一些资源的配合，因此，Nova compute 将会与 Nova volume、Nova network 等等一些组件通过消息队列的方式实现相互的协作。最终完成虚拟机实例的创建。

	![](http://i.imgur.com/2TuC6Da.png)
4. Nova 的功能特性：
	- 实现实例的生命周期的管理
	- 调动管理平台的网络、存储等资源
	- 提供了统一风格的 RestAPI 接口
	- 支持 KVM、VMware 等透明的 hypervisor
	- 各个模块之间通过消息队列来进行消息传递

##4、Swift 介绍
1. 什么是 Swift？
	- Swift 是 提供高可用分布式对象存储的服务，为 nova 组件提供虚拟机镜像存储服务，在数据冗余方面，无需采用 RAID 通过在软件层面，引入一致性散列技术和数据冗余，牺牲一定得数据一致性，来达到高可用和可伸缩性。
	- 支持多租户模式下，容器和对象读写操作，适用于互联网应用场景下非结构化的数据存储，比如，华为云盘等。
2. Swift 中的常用术语【1】
	- Account：用户定义的管理存储区域
	- Container：存储隔间，类似于文件夹或者目录
	- Object：包含了基本的存储尸体和它自身的元数据
	- Ring：环，记录了磁盘上存储的实体名称和物理位置的映射关系。
- 以上术语之间的关系：
	- 首先，可以创建多个 account
	- 每个 account 里可以创建多个容器 container
	- 每个 container 下可以创建多个 object，【container 之间不能相互嵌套】
- Swift 的功能
	- Swift 在物理结构上往往会存储对象的多个副本，通常按照物理位置的特点，将对象拷贝到不同的物理位置的特点，将对象拷贝到不同的物理位置上，来保证数据的可靠性。
- 常用术语 【2】
	- Region: 地域，从地理位置上划分的一个概念（基于灾备）
	- Zone：可用区，按照独立的供网、供电既基础设备划分
	- Node：节点，代表了一台存储服务器
	- Disk：磁盘，代表着物理服务器上的存储设备
	- Cluster：群集，为冗余考虑而设计的架构
- 以上术语之间的关系：
	- 可以根据不同的物理位置，有不同的 Region，不同的 region 代表两个不同的城市0然后在同一个 region 下，为冗余的考虑，设置了多个可用区，zone
	- 每一个 zone 可以有不同的存储节点 node
	- 在更大的架构上，两个 region 可以构成一个 cluster。
 
3. Swift 的架构
	- 首先，用户提出一个对象存储服务的申请，由 Swift 的ＡＰＩ接受和处理，收到之后，先去找 keystone 认证节点，对用户的身份进行认证
	- 认证通过后，将请求提交给，名称为 Swift Proxy 的组件，Swift Proxy 是 Swift 的代理，由 Swift Proxy 来确定究竟应该将存储对象放在哪一个满足存储要求的存储节点上
	- 最终将对象存储到指定的存储节点上即可。最终将返回结果返回给用户。

	![](http://i.imgur.com/20LLzPe.png)
##5、Keystone 介绍
- Keystone 介绍
	- 提供身份验证、服务规则和服务令牌功能
	- 任何服务之间相互调用，都需要经过 keystone 的身份验证
- 常用术语：
	- User：Openstack 最基本的用户
	- Project：指分配给使用者的资源的集合
	- Role：代表一组用户可以访问组员的权限
	- Domain：定义管理边界，可以包含多个 project/tenant、user、role 等
	- Endpoint：服务的 URL 路径，暴露出来的访问点
- Keystone 认证模型
	- 首先创建两个域 domain 1 和 2，给两个公司使用
	- 每个公司在 domain 下创建三个子公司 project 1 2 3
	- project 下可以创建多个用户 User，用户 User 可以跨多个 project 存在
- Keystone 认证原理
	- 当用户在创建时，将通过 Keystone 将会创建一个访问令牌 accesstoken
	- 假设当用户提出创建虚拟机实例的请求时，首先将自己的访问令牌和访问请求提交给 Nova 服务
	- Nova 服务为确保用户的访问令牌并没有篡改过，因此首先会将访问令牌交给 keystone 进行验证
	- 验证通过后 Nova 为了启动虚拟机的实例，还需要向 Glance 组件申请相关的镜像资源
	- Glance 为保证访问令牌在传递的过程中没有被篡改过，也需要将访问令牌发送给 keystone 做确认
	- 验证通过后将会发放镜像资源给 nova 组件
	- 当然，虚拟机实例的创建还需要存储、网络等资源，因此 nova 组件还需要给负责各种资源的模块传递申请资源的请求，资源申请的过程中都会伴随这访问令牌的验证
	- 最终，nova 拿到启动虚拟机实例的所有资源后进行实例的启动，然后分配给相关的用户
	- 整个过程来看组件之间资源的调用都离不开 keystone 的验证....

	![](http://i.imgur.com/blRKDLz.png)
##6、Neutron 介绍
- Neutron 的简介：
	- Neutron 是 open stack 中负责提供网络服务的组件，基于软件定义网络的思想，实现软件化的网络资源管理
	- 在实现上，充分利用了 linux 系统中各种与网络相关的技术，支持第三方插件
2. Neutron 中常用的术语
	- Bridge-int：综合网桥，常用于实现内部网路通讯功能的网桥
	- Br-ex: 外部网桥，通常用于跟外部网络通讯的网桥。
	- Neutron-server：提供了 API 接口，将配置好的 API 接口，提供给相关的插件，进行后续处理
	- Neutron-L2-agent: 二层代理，用于实现二层网络通讯的代理，用于管理 VLAN 的插件，接受 Neutron-server 的指令来创建 VLAN。
	- Neutron-DHCP－agent: 为子网自动分发 IP 地址
	- Neutron-L3-agent: 负责租户网络和 floating IP 之间的地址转换，通过 linux iptables 中的 NAT 功能来实现 IP 转换
	- Neutron-metadata-agent: 运行在网络节点上，用来响应 nova 组件中的 metadata 请求
	- LBaaS agent: 为堕胎实例和 open vswitch agent 提供负载均衡服务
3. Neutron 的架构
	- 当 Neutron 通过 API 接口，接受来自用户或者其他组件的网络请求时，以消息队列的方式提交给 2、3 层代理
	- 其中 Neutron-DHCP－agent 实现子网的创建和 IP 地址的自动分发
	- 而 Neutron-L2-agent 实现相同 VLAN 下，网络的通信
	- Neutron-L3-agent 实现同一个租户网络下，不同子网间的通信
	
	![](http://i.imgur.com/ixzea1j.png)
##7、Glance 介绍
1. Glance 的作用
	- 为 nova 提供镜像服务以便启动实例的组件
	- 但不负责镜像的本地存储，
	- 可以对镜像做快照、备份、镜像模板等管理
2. Glance 镜像支持的格式：
	- Raw
	- 经常被 vmware、visualbox 使用的 vhd
	- Vdi
	- 光盘 iso
	- openstack 经常使用的 qcow2
	- 亚马逊的 aki 和 ami
3. Glance 组件：
	- Glance-api 负责提供镜像服务的 rest api 服务, 作为镜像服务请求的入口。
	- Glance-registry 负责与 glance 使用的数据库交互，比如镜像的创建、删除、修改等操作
4. Glance 的架构：
	1. 当有来自 horizon、CLI、Nova、Compute 发送过来的镜像请求，由 glance api 接收处理，将请求的消息传递给 Glance-registry 组件
	2. 然后到数据库中查询镜像存储的位置信息，将查询到的结果返回给 api
	2. glance api 接下来将会调用 Storage adapter 组件进行查询，用来查询 glance 后端的存储，比如 SWIFT、Ceph、Amazon S3 等，最终获取镜像返回给用户。

	> glance api 也可以直接于数据库交互，但只传输少量数据

	![](http://i.imgur.com/NXdd2LB.png)
##8、Cinder 介绍
- Cinder 的简介
	- 为虚拟机实例提供 volume 卷的块存储服务，可将卷挂载到实例上，作为虚拟机实例的本地磁盘来使用
	2. 一个 volume 卷可以同时挂载到多个实例上
	3. 共享的卷同时只能被一个实例进行写操作，其他只能进行只读操作
 
- 支持的文件系统类型
	- LVM / ISCSI
	2. NFS
	3. NetAPP NFS
	1. Gluster
	5. DELL Equall Logic
 
- 常用术语
	- Volume 备份：volume 卷的备份，存放在备份的设备中
	- Volume 快照：卷在某个时间点的状态
	- Cinder API：为 Cinder 请求提供统一风格的 Rest API 服务，是 Cinder 服务的入口
	- Cinder Scheduler：负责为新建卷制定块存储设备
	- Cinder Volume：负责与存储的快设备交互，实现卷的创建、删除、修改等操作
	- Cinder Backup：备份服务负责通过驱动和后端的备份设备打交道。
 
- Cinder 架构
	- 当有用户或者 nova compute 提出创建卷的服务的请求时，首先由 Cinder API 接收请求，然后以消息队列的方式发送给 Cinder Scheduler 进行调用
	- Cinder Scheduler 侦听到来自 Cinder API 的消息队列后，到数据库中去查询当前存储节点的状态信息。并根据预定策略，选择卷的最佳 volume service 节点，然后将调度的结果发布出来，给 volume service 来调用
	- volume service 收到来自 volume schedule 的调度结果后会去查找 volume Provider。
	- volume Provider 在特定的存储节点上创建相关的卷，然后将相关的结果返回给用户，同时将修改的数据写入到数据库中。
![](http://i.imgur.com/51ST1wc.png)
##9、Ceilometer 介绍
- Ceilometer 的简介：
	- Ceilometer 是 open stack 中的一个子项目，为计费、监控等其他的服务提供数据支撑。
2. Ceilometer 存在的理由：
	- IaaS
	- 更多的公司利用 open stack 做自己的公有云平台。而作为共有云，计量和监控，这两个基础的服务往往是必不可少的，计量是为了获取平台中用户对自己的使用情况，监控是为了确保资源处于一个健康的状态
	- 因此，Ceilometer 在项目提出之初，是为了计量、计费而生。
3. Ceilometer 的核心概念
	- Ceilometer-agent-compute: 运行在计算节点上，是收集计算节点上信息的代理
	- Ceilometer-agent-central：运行在控制节点上，轮询服务的非持续化数据
	- Ceilometer-collector: 运行在一个或者多个控制节点上，监听 Message Bus【消息总线】，将收到的信息写入到数据库中
	- Storage: 数据存储，支持 mongo DB，mysql 等等。用于存储收集到的样本数据
	- API server: 运行在控制节点上，提供对数据库的数据的访问
	- Message Bus: 计量数据的消息总线，收集数据给 Ceilometer-collector
4. Ceilometer 架构
	- Ceilometer 采用了两种数据采集的方式，其中一种是消费了 openstack 内各个服务自动发出的 notification 消息，【图中的蓝色箭头】，另外一种是调用各个服务的 API，去主动轮询获取数据。【图中的黑色箭头】
 ![](http://i.imgur.com/5YzoG2G.png)
- 为什么采用两种数据采集的方式？【也是工作架构】
	- 第一种:
		- 因为在 openstack 中，大部分事件都会发出 notification 消息
		- 比如创建删除 instance 实例的时候，这些计量计费的信息时，都会发出 notification 消息
		- 而作为 Ceilometer 组件，就是 notification 消息的最大的消费者
		- 因此，第一种方式，是 Ceilometer 的首要的数据来源。
	- 第二种:
		- 但是，也有一些计量的消息，是 notification 获取不到的，比如一些 instance 的 CPU 的运行时间，或者是 CPU 的使用率等等
		- 因此，Ceilometer 增加了第二种方式，即为周期性的调用相关的 API，去轮询这些消息。

##10、Heat 介绍
- Heat 简介
	- Heat 是一个基于模板来创建相关资源的服务，是 OpenStack 核心项目之一。可以通过. yaml 文件生成模板，通过 Heat-agent 组件在 OpenStack 中创建相关的资源。
	- 除此之外，支持自由的 Haut 模板。以及亚马逊的 cloudy formation 格式的模板。
	- 模板支持丰富的资源类型，不仅支持了常用的基础架构，包括计算、网络、存储、 镜像等功能，还包括了 Ceilometer 警报等高级资源。
	- 提供基于模板的编排服务。
2. 常用术语
	- Stack：Heat 要用到的所有设施和资源的集合。
	- Heat template：是以. yaml 结尾的文件，用于创建 stack。
	- Heat-api：提供 rest api 服务，Heat 入口将 api 请求发送给 heat engine 去执行。
	- Heat-api-cfn：支持亚马逊格式访问的 rest api。
	- Heat-engine：Heat 的核心模块，接收 API 请求，在 openstack 中创建资源。
	- Heat-cfntools、Heat-init：在镜像中安装完成虚拟实例操作任务的工具。
	- Heat-api-cloudwatch： 监控编排服务。
	- Resource：底层各种服务抽象的集合。（例：计算、存储、网络）
	- Heat-client：用于调用访问其它各个组件的 client 工具
3. Heat 架构

	![](http://i.imgur.com/HFUhcTu.png)

	1. 当用户在 Horizon 中或者命令行（CLI）中提交包含模板和参数的，创建实例的请求时，Heat 服务接收请求，调用 Heat-API/Heat-API-cfn
	- 然后 API/API-cfn 首先验证模板的正确性，然后通过消息队列异步传输给 Heat Engine 进行处理。

	![](http://i.imgur.com/sSVyPOP.png)

	1. 当 Heat Engine 拿到请求后，会把请求解析为各种资源类型，而每一种资源都有相关的 Client 与相关的服务对应。
	- Client 通过发送 rest 请求给其它的服务，从而获取相关的资源，最终完成请求的处理。
	
- 在整个过程中，Heat Engine 的作用分为三层：
	1. 处理 Heat 层面的请求，然后根据模板和输入的参数来创建 Stack。
	- 解析 Stack 中各种资源的依赖关系以及 Stack 的嵌套关系。
	- 根据解析出来的关系，依次调用各种服务的 Clinet，来创建各种资源。

##test
	下列关于 Nova 组件各个功能模块的描述哪一项是错误的？A
	
	   A  nova-scheduler 直接访问数据库从而决定将实例分配到具体的计算节点上
	   B  nova-conductor 负责提供 nova 数据库连接的服务
	   C  nova-compute 是运算在计算节点上的服务，主要用于创建和管理虚拟机实例
	   D  nova-api 模块能提供 rest 风格的 API 服务接口
	
	Openstack 组件中下列哪一项可以以云盘的存储方式对外提供服务？A
	
	   A  swift
	   B  celiometer
	   C  cinder
	   D  glance
	
	Openstack 的开源项目最早是由 rackspace 公司和 NASA 提供的，该两家机构提供项目对应正确的是下列哪一项？A
	
	   A  swift 和 nova
	   B  neutron 和 nova
	   C  cinder 和 nova
	   D  swift 和 neutron
	
	) 下列关于虚拟机实例管理中网络配置的描述，哪些是正确的？D
	
	   A  指定租户内，普通_member_角色下用户可以创建私有网络并指定子网的网段
	   B  指定租户内，普通_member_角色下用户可以为自己创建的实例绑定 floating IP 地址
	   C  指定租户内，普通_member_角色下用户申请 floating IP 地址
	   D  指定租户内，普通_member_角色下用户可以创建外网网络并指定外网的网段地址
	
	下列关于创建虚拟机实例时指定镜像位置的描述错误的有哪些？AD
	
	   A  通过 --copy-from 指定一个 NFS 或 SMB 的共享路径
	   B  通过 --file 指定本地系统的位置
	   C  通过 --location 指定一个 URL 路径
	   D  所有说法都正确