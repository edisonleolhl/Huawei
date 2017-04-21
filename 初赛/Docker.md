#三、Docker
> 本文包括：

> 1、Docker 发展历程

> 2、容器技术精髓剖析

> 3、Docker 核心技术

> 4、Docker 平台架构

> 5、Docker 平台的对比

> 6、Docker 生态圈及企业应用案例
##1、Docker 发展历程
1. 容器和 Docker
	- 容器：在 linux 中，容器技术是一种进程隔离的技术，应用可以运行在每个相互隔离的容器中。
	- 容器与虚拟机的区别：在容器中，各个应用共用一个 kernel
	- Docker：Docker 是一家公司，在 2013 年之前公司名叫 dotCloud，Docker 仅仅是一个容器管理的产品。在 13 年，将 Docker 开源，Docker 风靡全球，公司也更名为 Docker。
2. 容器技术的演进
	- 1979 年，有了容器技术的雏形，root 技术的引进开启了进程隔离技术
	- 2000 年，FreeBSD Jails 将计算机分为多个独立的小型计算系统。
	- 2006 年，谷歌 Process Containers 技术，在进程隔离的基础上，进行了计算资源的限制
	- 2018 年，LXC，第一个完整的容器管理工具
	- 2013 年，LMCTFY 实现了 linux 应用程序的程序化，成为 libcontainer 的重要组成部分。
	- 2013 年，Docker 最初使用的是 LXC，后来被替换成 libcontainer 
3. Docker 技术的迅猛发展
	- 在开源之后迎来迅猛的发展，在现在也保持着迅猛的发展势头 
4. Docker 技术发展迅猛的原因总结：
	1. 应用架构正在发生变革 --- 微服务化
		- 在互联网时代，为了实现更快的开发迭代和更好的弹性伸缩，互联网应用不再采用传统的 3 层架构，而是采用微服务的方式，解耦，快速交付
	2. 基础架构系统也在发生变革 --- 虚拟化、混合云
		- 从硬件服务器到虚拟机到企业私有云，从本地数据中心到灾备数据中心再到公有云。
5. 他山之石，可以攻玉【容器技术的作用】
	- 一次封装，多次部署，随时迁移，不需要关注底层环境 
6. Docker 定义的标准 + 服务应用
	- 基础设施标准化（Docker Engine）：有了 Docker Engine，可将 Docker 容器跑起来
	- 应用交付的标准化（Docker Image）: 提供了一套应用快速打包为轻量级 Docker Image 的方法。开发人员在代码完成之后，可以将其打包为镜像
	- 运维管理的标准化（Docker container）： 运维人员不在需要将应用准备系统、运行环境、组件和基础软件包, 容器时代，应用都运行在一个个的 Docker container 中。标准运维将关注容器，而不是复杂的系统环境。
	- 分发部署标准化（Docker Registry）：指的是容器化之后不同版本的应用镜像都存储在镜像仓库中。

##2、容器技术精髓剖析
1. namespace 技术，包含了六项隔离，下面是 namespace 及其对应的隔离内容：
	- UTS ——主机名与域名
	- IPC ——通信（信号量、消息队列和共享内容）
	- PID ——进程编号，通过 PID 技术，同个主机上的不同容器可以有相同的 PID 进程。
	- Network ——网络设备、网络栈、端口
	- Mount ——让文件拥有自己的文件系统
	- User ——用户和用户组，实现用户权限的隔离
2. cgroups 技术
	- 通过 cgroups 可以为容器设定系统资源的配额，包括 CPU、内存、I/O 等等。
	- 对于不同的系统资源，cgroups 提供了统一的接口，对资源进行控制和统计。
	- 限制的具体方式不尽相同，实际的流程很复杂。
3. 其它相关 Linux Kernel 技术
	- selinux 和 apparmor：增强对容器的访问控制
	- capabilities：将超级用户 root 的权限分割成各种不同的 capability 权限，从而更严格地控制容器的权限。
	- netlink：完成 Docker 的网络环境配置和创建
	- 这些技术从安全、隔离、防火墙、访问等方面为容器的成熟落地打下了坚实的基础。
4. 容器管理
	1.  lxc
		- 是第一个完整意义上的容器管理技术。通过 lxc 可以方便的创建、启动和停止一个容器。
		- 还可以通过 lxc 来操纵容器中的应用，也可以查看容器的运行状态。
		- Docker 的出现把 2008 年的 lxc 的复杂的使用方式简化为自己的一套命令体系。
	2. libcontainer
		- Docker 后来开发了原生的 libcontainer 代替了 lxc。
		- libcontainer 实际上反向定义了一组接口标准。
		- 反向定义：libcontainer 并不是为了调用底层的 Linux Kernel 技术而设计的，而是 Linux Kernel 技术符合了定义出来的 libcontainer 标准，Docker 引擎才能运行起来。如果此后还有新技术符合这套标准，Docker 引擎还是可以正常运行。
		- 这样的设计思路为 Docker 的跨平台实现和全面化应用带来了可能。
5. Docker 技术原理
	- Docker 构造：Client-Server

	- Docker 结构
		- 装好了 Docker 工具之后，也就同时装好了 Client 端和 Server 端。
		- Client 端可以是 Docker 命令行工具，也可以是 GitHub 上开源的图形化工具。通过 Client 工具可以发起创建、管理容器的指令到 Server 端。

- Docker Daemon
	- Docker Daemon 通过 libcontainer、lxc 的技术来完成容器管理操作。
	- Docker Daemon 的三个重要组件：
		- execdriver：存储了容器定义的配置信息，libcontainer 拿到配置信息以后调用底层的 namespace 等技术来完成容器的创建和管理。
		- networkdriver：完成容器网络环境的配置，包括了容器的 IP 地址、端口、防火墙策略，以及与主机的端口映射等。
		- graphdriver: 负责对容器镜像的管理。

##3、Docker 核心技术
1. VM Vs. Docker
	- 应用在 Docker 中跑起来是什么样的？
	- Docker 和虚拟机的结构对比
		- 虚拟机下，是硬件→操作系统→Hypervisor→虚拟机操作系统→配置虚拟机依赖环境→安装 App
		- 容器时代，硬件→操作系统→Docker Engine→运行所需要的依赖环境→安装 App
		- 容器的运行不需要安装虚拟机的操作系统，是比虚拟机更加轻量的虚拟化技术。
	- Docker 和虚拟机对比的详细信息
		- Docker 容器共用一个 Kernel，而虚拟机使用自己操作系统的 Kernel→虚拟机拥有比 Docker 更好的隔离性。
		- Docker 有更多的优势：
			- 虚拟机操作系统的存在，占用了更多的计算资源；
			- 空间占用上，虚拟机是 GB 级，Docker 可以小至几 MB；
			- Docker 的启动时间更快（毫秒级）；
			- Docker 有更强的快速扩展能力、跨平台迁移能力（虚拟机无法从VMware 迁移到 KVM）
2. Docker 的重要概念：容器、镜像、仓库
	- 容器：与虚拟机一样都是承载相关应用的载体
		- Docker 容器的状态
			- 三种状态：Running、Stopped 和 Paused，与虚拟机的运行、关机、挂起状态相似。
	- 镜像：当容器安装了特定的应用之后，就可以将其打包成镜像。需要应用的时候，将镜像下载下来，就可以快速运行起来。
		- 镜像的生成
			- 当把镜像下载到本地之后，可以使用 Docker run 的命令，启动一个基于这个镜像的容器。对容器修改之后，可以将其 commit 回去，生成一个新版本的镜像。
			- Docker 镜像是一种层级结构的文件系统，最上层往往是可写的，存储了已经运行的容器的修改信息当对容器进行 kill 的时候，修改信息就会被删掉。当容器被 commit 成镜像的时候，这些修改信息也会保存成新的层级。
			- 镜像的生成除了使用 commit 之外，还可以使用 Dockerfile（更标准、更常用）。这种方式 build 生成出来的镜像更加干净、透明。
	- 仓库：存储镜像的地方。
		- Dockerhub 和私有仓库
			- Dockerhub 是 Docker 的官方仓库，存放着各种官方的标准镜像。可以使用 pull 命令直接从 Dockerhub 中下载镜像到本地进行使用。
			- 还可以构建自己的镜像仓库，用于存放常用的镜像以及企业自定义的应用镜像。可以从私有仓库中下载、上传镜像。
3. Docker 核心技术：Build, Ship, Run
	- Docker 的主要操作
		- 首先利用 Dockerfile 将组建 Build 成为一个镜像，然后将镜像上传到企业自定义的镜像仓库中。
		- 当我们需要镜像的时候，可以从任何地方连接到镜像仓库中，将镜像下载到本地，然后一键将其 Run 起来。
		- 这就是 "Build once, run everywhere"：一次构建，任何地方都能运行。
4. Docker 数据卷
	- 保存 Docker 数据
		- 容器一旦关闭，修改信息就会丢失，这对于一些有状态的应用来说往往是不可接受的。
		- 可以通过给文件挂载文件目录或者存储来解决，从而可以存储容器运行中的一些数据。这样，当容器崩溃，重启容器的时候，依然可以访问之前容器存储下来的一些数据。
		- 这种方式也可以解决一些主机和容器之间的数据访问。
5. Docker 网络
	- 实现容器之间的通信、容器与外部之间的通信。
	- 四种模式：
		- Bridged：表示容器可以与主机上的容器，主机上的外部进行通信
		- Host：表示容器只能与主机通信
		- Container：表示容器只能与容器通信
		- None：没有网络连接
	- 比较常用的是 Birdged 模式。主机会生成一个 Docker 网桥，每个容器可以拥有自己的虚拟网卡，容器网卡通过网桥连接到主机的物理网卡，与外部进行通信。

##4、Docker 平台架构
1. 容器编排（orchestration）
	- 这里的编排泛指广义的编排，用于管理容器下面的主机，管理容器以及日期之间的逻辑关系，即为我们所知的应用架构。
	- 集群管理：【关键词：配置管理、资源视图、节点增删、高可用】
	- 容器调度：【关键词：容器部署、调度策略、互斥】
	- 故障恢复：【关键词：主机检查、容器检查】
	- 应用编排：应用是一个个细微的容器来组成，每一个容器之间具有一定的逻辑关系，我们需要使用简单明了的编排语句将这些容器关联起来，比如容器之间的端口访问，容器的启动顺序等，从而实现整个应用的逻辑架构。
2. 编排主流的 3 大工具
	- Swarm: Docker  2014 年发布 内置于 Docker，和 compose 一起使用
	- Mesos: Apache  2007 年发布 一般会结合 marathon、Aookeeper 一起使用
	- Kubernets: Google 2014 年发布 结合 Etcd 一起使用
3. 负载均衡和服务发现
	- 负载均衡：请求到达负载均衡器之后，负载均衡器平均的分配到后面的容器上。
		- 常用的负载均衡的技术：haproxy,LVS,F5,Nginx
	- 服务发现：服务发现会自动将容器的配置信息上传至配置中心（config center），包含了容器的 IP、 端口、对外的域名等。
		- 常用的服务发现的技术：Etcd,Zookeeper,Consul
	- 负载均衡器会周期性的从配置中心获取配置信息，并且将容器加入到相关的负载均衡访问架构中
4. 日志管理
	- 日志：包含主机、编排工具、日志、容器、容器中的应用等等相关的日志
	- 对日志处理平台的要求：集中化、海量存储、灵活过滤、快速查询、伸缩性架构、高可用、强大的 UI
	- 日志管理软件：ELK  ， 包含了 3 个组件:
		- Logstash，用于收集各种各样的日志
		- ElasticSearch: 主要用于存储和搜索日志
		- Kbana: 用于界面展示的管理工具
5. Docker 监控
	- 包括 主机 镜像 容器 应用等维度进行监控
	- 构建相关的告警, 跟踪，监控等监控流程体系，即 WANT 原则(Watching(监控)、Answer（响应）、Notify（通知）、Track（跟踪），总结起来就是 WANT)
	- 常用的监控工具：Zabbix, Nagios, cAdvisor, Datadog, Scout 等
6. Docker 平台架构
	- 平台层：底层需要对计算，网络，存储等资源进行管理，结合容器引擎和容器编排的工具实现对容器化应用的落地
	- 能力层：需要实现容器化应用的弹性架构，负载均衡等等，需要实现平台的统一监控和统一的日志管理，需要有严格的权限体系实现对用户，租户的管理，需要结合当前的 deverups 理念来实现对应用的持续交付和版本控制
	- 最上层：平台还需提供应用的 Web UI 和功能齐全的 API 服务
7. Docker 平台技术体系
	- Orchestration(编排): Swaim, Kubernetes, Mesos
	- Cluster Management(集群管理): Etcd, Zookeeper, Haproxy
	- CI/CD(持续集成): Jenkins, GitLab, Registry
	- Database(数据库): redis, MySQL, mongoDB
	- ELK&CMDB(日子和配置管理): Logstash, Chef, puppet
	- Monitor(监控): ZABBIX, Nagios, sysdig

##5、Docker 平台的对比
1. Mesos 介绍
	- 两个重要角色，一个是 Slave, 安装集群节点上的，还有  Master, 作为集群的管理节点，slave 会将节点的资源使用情况周期性的报告给我们的 master 节点。
	- Mesos 需要配合架构的 FrameWork 进行资源调度，过程如下：
		- master 会定期将计算机节点的资源使用情况周期性的报告给我们的 Framework Scheduler
		- Framework Scheduler 进行调配后，下发部署的任务给集群节点
		- 节点上的 framework Executor 获取任务进行容器的部署（在容器的架构中，Executor 就是容器的引擎）
		- 节点会将部署结果反馈给 master
		- Master 会更新主机资源的状态给 Framework Scheduler
	- Mesos+Marathon+Zookeeper
		- Marathon 是一个可以调用 Docker 引擎的 framework，可以将容器按照一定的调度策略部署到合适的主机上。
		- Zookeeper 的作用是保证 Marathon 和 Mesos 来管理节点的高可用性，即当 master 节点宕机之后，可以快速的选取出新的 master 节点，从而不影响逻辑架构。Zookeeper 本身是一个分布式的高可用的架构 
2. kubernetes 的介绍（Goolge 的开源容器管理项目）
	- 重点：mesos 中的最小的单元是容器，但是 kubernetes 的最小的单元是 Pod。容器被封装在 Pod 中，一个 Pod 中可以存放一个或者多个容器。
	- 设计 Pod 的目的：将需要紧密联系的 Docker 容器放置在一个独立的空间（Pod）内。
		- 同个 pod 中的容器可方便的共享存储
		- 同个 pod 中的容器可直接访问另一个容器
	- kubernetes 整体架构：在部署 kubernetes 之前，需要先部署 Etcd 作为集群的管理工具
	- 2 个重要的角色：minion（普通节点）和 master（管理节点）
3. Swarm 的介绍（Docker 公司自己的容器管理工具）
	- 1.12 版本之后，Swarm 已经封装进 Docker 引擎中，并且自带服务发现的功能
	- 2 个重要角色：manager node 和 worker node
	- Swarm1.12 之后，自带服务发现的功能，可以做到 manager node 的高可用性。
	- Swarm 也内置了负载均衡的技术，使用的是 LVS，但是依然支持和其他服务发现的工具，如 ETCD，Haproxy 
4. Docker 平台对比【表格无法显示】
	![](http://i.imgur.com/0Jbs12d.png)

##6、Docker 生态圈及企业应用案例
1. 应用场景：快速交付与 CICD
	- 企业应用的开发上线流程一般是：代码、构建编译、测试、发布、部署
	- 遇到的问题：可能因为环境的问题导致上线延迟，测试不通过等。
	- 快速交付：Docker，通过 Docker 可以大大的提高环境交付的质量和速度，开发人员写好代码之后，交付的不在是一大堆的附属文档，而是一个个的镜像存储到镜像仓库中。在测试环境、预生产环境以及生产环境将镜像仓库中的镜像拉取出来即可。保证部署出来的所有应用都是标准的、统一的。即为实现了应用的快速交付。
	- CICD：持续集成和持续部署（Constant Integration Constant Deployment）当我们的代码更新时，开发人员可以构建一个新的镜像版本到镜像仓库中，运维人员可以快速的将我们的镜像应用到测试环境、预生产环境以及生产环境。甚至可以通过金 KISS 实现整个更新的自动化，从而实现了持续集成持续部署，实现了应用开发环境的快速迭代。
2. 应用场景：云间迁移
	- 应用容器化之后，对底层环境的要求将大大的降低，应用可以实现从本地数据中心到 ALS，阿里云、公有云等迁移
3. 应用场景：弹性扩展
	- 企业应用容器化之后，应用的扩展就是拉取镜像部署更多的容器的简单的过程，我们可以部署相关的监控系统，当发现应用访问慢或者是资源紧张的时候，在弹性扩展的策略下，应用会自动增加相应的容器实例，从而减轻应用访问的压力。当集群中的主机资源不足的时候，还可以使用 IaaS 接口，自动的增加主机的数量，以便于创建更多的 Docker 容器。
4. 应用案例：
	- 平安 Padis 平台
	- 京东 618（基于 openstack 和 Docker）
	- 天猫双十一
5. Docker 巨大生态势能
	- 从安全架构领域，操作系统领域、网络、存储、安全、安全、监控、日志等方面，越来越多的公司卷入到 Dockers 的发展潮流当中。
6. 基于 Docker 的产品：
	- 红帽 openshift
	8. 阿里云容器服务
	9. Azure 容器服务
	10. 网易蜂巢
	11. 道客云、有容云、希云、时速云等

##test
	Docker 容器本质上是宿主机上的进程，宿主机只能是硬件服务器。B
	
	   A  对
	   B  错
	
	下列哪个不是 Docker 引擎的组件？C
	
	   A  Docker daemon
	   B  Docker CLI
	   C  Docker registries
	   D  Rest API 接口
	
	下列关于 Docker 的 namespace 特征特性描述中哪一项是不正确的？C
	
	   A  不同用户的进程通过 pid namespace 隔离开
	   B  通过 net namespace 实现网络隔离
	   C  namespace 不允许每个 container 拥有独立的 hostname 和 domain name
	   D  mnt namespace 允许不同 namespace 的进程看到的文件结构不同
	
	在 Docker 中，关于容器的描述中不正确的哪项？D
	
	   A  Docker 利用容器来运行应用
	   B  容器看做是一个简易版的 Linux 环境
	   C  每个容器都是相互隔离的
	   D  Docker 容器比较适合运行多个进程
	
	 Docker 作为成功的容器解决方案，具有下列哪些特性？ACD
	
	   A  简单易用
	   B  无需维护
	   C  跨平台
	   D  可移植