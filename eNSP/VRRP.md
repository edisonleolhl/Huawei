# VRRP

## VRRP 简介

- 虚拟路由冗余协议 VRRP（Virtual Router Redundancy Protocol）通过把几台路由设备联合组成一台虚拟的路由设备，将虚拟路由设备的 IP 地址作为用户的默认网关实现与外部网络通信。当网关设备发生故障时，VRRP 机制能够选举新的网关设备承担数据流量，从而保障网络的可靠通信。
- 随着网络的快速普及和相关应用的日益深入，各种增值业务（如 IPTV、视频会议等）已经开始广泛部署，基础网络的可靠性日益成为用户关注的焦点，能够保证网络传输不中断对于终端用户非常重要。
- 通常，同一网段内的所有主机上都设置一条相同的、以网关为下一跳的缺省路由。主机发往其他网段的报文将通过缺省路由发往网关，再由网关进行转发，从而实现主机与外部网络的通信。
- 当网关发生故障时，本网段内所有以网关为缺省路由的主机将无法与外部网络通信。增加出口网关是提高系统可靠性的常见方法，此时如何在多个出口之间进行选路就成为需要解决的问题。
- VRRP 的出现很好的解决了这个问题。VRRP 能够在不改变组网的情况下，采用将多台路由设备组成一个虚拟路由器，通过配置虚拟路由器的 IP 地址为默认网关，实现默认网关的备份。当网关设备发生故障时，VRRP 机制能够选举新的网关设备承担数据流量，从而保障网络的可靠通信。
- 在具有多播或广播能力的局域网（如以太网）中，借助 VRRP 能在网关设备出现故障时仍然提供高可靠的缺省链路，无需修改主机及网关设备的配置信息便可有效避免单一链路发生故障后的网络中断问题。

## 原理

![alt](http://www.023wg.com/content/uploadfile/201512/b5a71449480556.png)

- 如上图 1 所示，HostA 通过 Switch 双归属到 SwitchA 和 SwitchB。在 SwitchA 和 SwitchB 上配置 VRRP 备份组，对外体现为一台虚拟路由器，实现链路冗余备份。
- 我们可以在如上图 1 所示的网络中部署 VRRP 协议，下面结合该图介绍 VRRP 协议的基本概念：
    1. VRRP 路由器（VRRP Router）：
    运行 VRRP 协议的设备，它可能属于一个或多个虚拟路由器，如 SwitchA 和 SwitchB。
    1. 虚拟路由器（Virtual Router）：
    又称 VRRP 备份组，由一个 Master 设备和多个 Backup 设备组成，被当作一个共享局域网内主机的缺省网关。如 SwitchA 和 SwitchB 共同组成了一个虚拟路由器。
    1. Master 路由器（Virtual Router Master）：
    承担转发报文任务的 VRRP 设备，如 SwitchA。
    1. Backup 路由器（Virtual Router Backup）：
    一组没有承担转发任务的 VRRP 设备，当 Master 设备出现故障时，它们将通过竞选成为新的 Master 设备，如 SwitchB。
    1. VRID：
    虚拟路由器的标识。如 SwitchA 和 SwitchB 组成的虚拟路由器的 VRID 为 1。
    1. 虚拟 IP 地址 (Virtual IP Address)：
    虚拟路由器的 IP 地址，一个虚拟路由器可以有一个或多个 IP 地址，由用户配置。如 SwitchA 和 SwitchB 组成的虚拟路由器的虚拟 IP 地址为 10.1.1.10/24。
    1. IP 地址拥有者（IP Address Owner）：
    如果一个 VRRP 设备将虚拟路由器 IP 地址作为真实的接口地址，则该设备被称为 IP 地址拥有者。如果 IP 地址拥有者是可用的，通常它将成为 Master。
    如 SwitchA，其接口的 IP 地址与虚拟路由器的 IP 地址相同，均为 10.1.1.10/24，因此它是这个 VRRP 备份组的 IP 地址拥有者。
    1. 虚拟 MAC 地址（Virtual MAC Address）：
    虚拟路由器根据虚拟路由器 ID 生成的 MAC 地址。一个虚拟路由器拥有一个虚拟 MAC 地址，格式为：00-00-5E-00-01-{VRID}(VRRP for IPv4)；00-00-5E-00-02-{VRID}(VRRP for IPv6)。
    当虚拟路由器回应 ARP 请求时，使用虚拟 MAC 地址，而不是接口的真实 MAC 地址。如 SwitchA 和 SwitchB 组成的虚拟路由器的 VRID 为 1，因此这个 VRRP 备份组的 MAC 地址为 00-00-5E-00-01-01。
- VRRP 状态机
  - VRRP 协议中定义了三种状态机：初始状态（Initialize）、活动状态（Master）、备份状态（Backup）。其中，只有处于 Master 状态的设备才可以转发那些发送到虚拟 IP 地址的报文。
  - Initialize：
    - 该状态为 VRRP 不可用状态，在此状态时设备不会对 VRRP 报文做任何处理。
    - 通常刚配置 VRRP 时或设备检测到故障时会进入 Initialize 状态。
    - 收到接口 Up 的消息后，如果设备的优先级为 255，则直接成为 Master 设备；如果设备的优先级小于 255，则会先切换至 Backup 状态。
  - Master：
    - 定时（Advertisement Interval）发送 VRRP 通告报文。
    - 以虚拟 MAC 地址响应对虚拟 IP 地址的 ARP 请求。
    - 转发目的 MAC 地址为虚拟 MAC 地址的 IP 报文。
    - 如果它是这个虚拟 IP 地址的拥有者，则接收目的 IP 地址为这个虚拟 IP 地址的 IP 报文。否则，丢弃这个 IP 报文。
    - 如果收到比自己优先级大的报文，立即成为 Backup。
    - 如果收到与自己优先级相等的 VRRP 报文且本地接口 IP 地址小于对端接口 IP，立即成为 Backup。
  - Backup：
    - 接收 Master 设备发送的 VRRP 通告报文，判断 Master 设备的状态是否正常。
    - 对虚拟 IP 地址的 ARP 请求，不做响应。
    - 丢弃目的 IP 地址为虚拟 IP 地址的 IP 报文。
    - 如果收到优先级和自己相同或者比自己大的报文，则重置 Master_Down_Interval 定时器，不进一步比较 IP 地址。
- VRRP 工作过程
    - VRRP 的工作过程如下：
    - VRRP 备份组中的设备根据优先级选举出 Master。Master 设备通过发送免费 ARP 报文，将虚拟 MAC 地址通知给与它连接的设备或者主机，从而承担报文转发任务。
    - Master 设备周期性向备份组内所有 Backup 设备发送 VRRP 通告报文，以公布其配置信息（优先级等）和工作状况。
    - 如果 Master 设备出现故障，VRRP 备份组中的 Backup 设备将根据优先级重新选举新的 Master。
    - VRRP 备份组状态切换时，Master 设备由一台设备切换为另外一台设备，新的 Master 设备会立即发送携带虚拟路由器的虚拟 MAC 地址和虚拟 IP 地址信息的免费 ARP 报文，刷新与它连接的主机或设备中的 MAC 表项，从而把用户流量引到新的 Master 设备上来，整个过程对用户完全透明。
    - 原 Master 设备故障恢复时，若该设备为 IP 地址拥有者（优先级为 255），将直接切换至 Master 状态。若该设备优先级小于 255，将首先切换至 Backup 状态，且其优先级恢复为故障前配置的优先级。
    - Backup 设备的优先级高于 Master 设备时，由 Backup 设备的工作方式（抢占方式和非抢占方式）决定是否重新选举 Master。 
    - 抢占模式：
      - 在抢占模式下，如果 Backup 设备的优先级比当前 Master 设备的优先级高，则主动将自己切换成 Master。
    - 非抢占模式：
      - 在非抢占模式下，只要 Master 设备没有出现故障，Backup 设备即使随后被配置了更高的优先级也不会成为 Master 设备。
    - 由此可见，为了保证 Master 设备和 Backup 设备能够协调工作，VRRP 需要实现以下功能：
      - Master 设备的选举；
      - Master 设备状态的通告。
    - 下面将从上述两个方面详细介绍 VRRP 的工作过程。
    - [http://www.023wg.com/kkxpz/79.html](http://www.023wg.com/kkxpz/79.html)

## 功能

### 主备备份

![alt](http://www.023wg.com/content/uploadfile/201512/c37a1449564259.png)

- 主备备份是 VRRP 提供备份功能的基本方式。
- 如上图所示。该方式需要建立一个虚拟路由器，该虚拟路由器包括一个 Master 设备和若干 Backup 设备。
  - 正常情况下，SwitchA 为 Master 设备并承担业务转发任务，SwitchB 和 SwitchC 为 Backup 设备且不承担业务转发。
  - SwitchA 定期发送 VRRP 通告报文通知 SwitchB 和 SwitchC 自己工作正常。如果 SwitchA 发生故障，SwitchB 和 SwitchC 会根据优先级选举新的 Master 设备，继续为主机转发数据，实现网关备份的功能。
  - SwitchA 故障恢复后，在抢占方式下，将重新选举成为 Master；在非抢占方式下，将保持在 Backup 状态。
- 下面以上图为例简要说明 VRRP 主备备份的基本原理。
  - SwitchA 为 Master 设备，优先级设置为 120，抢占方式为延迟抢占。
  - SwitchB 为 Backup 设备，优先级为默认值 100，抢占方式为立即抢占。
  - SwitchC 为 Backup 设备，优先级设置为 110，抢占方式为立即抢占。
- 正常情况下，用户侧的上行流量路径为：Switch->SwitchA->Router。此时，SwitchA 定期发送 VRRP 报文通知 SwitchB 和 SwitchC 自己工作正常。
- 当 SwitchA 发生故障时，SwitchA 上的 VRRP 会处于不可用状态。由于 SwitchC 优先级高于 SwitchB，因此 SwitchC 变为 Master 设备，并开始发送 VRRP 报文和免费 ARP 报文，SwitchB 继续保持为 Backup 设备。用户侧的上行流量路径为：Switch->SwitchC->Router。
- 当 SwitchA 故障恢复时，VRRP 的优先级为 120，状态变为 Backup。此时 SwitchC 继续定期发送 VRRP 报文，当 SwitchA 收到 VRRP 报文后，会比较优先级，发现自己的优先级更高，等待抢占延迟后抢占为 Master 设备，并开始发送 VRRP 报文和免费 ARP 报文。用户侧的上行流量路径恢复为：Switch->SwitchA->Router。

### VRRP 负载分担

- 负载分担方式是指多台设备同时承担业务，因此负载分担方式需要两个或者两个以上的虚拟路由器，每个虚拟路由器都包括一个 Master 路由器和若干个 Backup 路由器，各虚拟路由器的 Master 路由器可以各不相同。
- VRRP 负载分担与 VRRP 主备备份的基本原理和报文协商过程都是相同的。VRRP 负载分担与 VRRP 主备备份方式不同点在于：
- 负载分担方式需要建立多个 VRRP 备份组，各备份组的 Master 设备可以不同；
- 同一台 VRRP 设备可以加入多个备份组，在不同的备份组中具有不同的优先级。
- 在网关设备上配置 VRRP 主备备份功能，可以很方便的实现网关的冗余备份。为减轻主用设备上数据流量的承载压力，用户可以通过配置 VRRP 负载分担实现上行流量的负载均衡。
- 通过创建多个带虚拟 IP 地址的 VRRP 备份组，为不同的用户指定不同的 VRRP 备份组作为网关，实现负载分担。

![alt](http://www.023wg.com/content/uploadfile/201512/21ac1449564259.png)

- 如上图所示，配置两个 VRRP 备份组。
- VRRP 备份组 1：SwitchA 为 Master 设备，SwitchB 为 Backup 设备。
- VRRP 备份组 2：SwitchB 为 Master 设备，SwitchA 为 Backup 设备。
- 一部分用户将 VRRP 备份组 1 作为网关，另一部分用户将 VRRP 备份组 2 作为网关。这样即可实现对业务流量的负载分担，同时，也起到了相互备份的作用。

## 配置实例

- 在交换机上部署 VRRP 功能时需注意：
  1. 不同备份组之间的虚拟 IP 地址不能重复，并且必须和接口的 IP 地址在同一网段。
  1. 保证同一备份组的设备上配置相同的 virtual-router-id。
  1. 不同 VLANIF 接口上可以绑定相同 virtual-router-id 的 VRRP 备份组，但不同以太网接口上不可以绑定相同 virtual-router-id 的 VRRP 备份组。
  1. 在设备上同时配置 VRRP 和静态 ARP 时，需要注意：当在 VLANIF 接口、以太网接口下配置 VRRP 时，不能将与这些接口相关的静态 ARP 表项对应的映射 IP 地址作为 VRRP 的虚拟地址。否则会生成错误的主机路由，影响设备之间的正常转发。
  1. 不能将 VRRP 的虚拟 MAC 地址配置为静态 MAC 地址或者黑洞 MAC 地址。
  1. 如果 VRRP 备份组内各交换机上配置的 VRRP 协议版本不同，可能导致 VRRP 报文不能互通。
- 创建 VRRP 备份组:
  - 在 vrrp 接口视图（即 vrrp 设备的下行接口，可以是物理接口、逻辑接口或子接口）下设置

        [Huawei-GigabitEthernet0/0/1]vrrp vrid 1 virtual-ip 10.1.1.1
  - vrid：为 vrrp 备份组号
  - virtual-ip：指所创建的 vrrp 备份组的虚拟 IP 地址，该虚拟 IP 地址必须和对应接口的真实 IP 地址在同一段；还可以为同一个备份组设置多个虚拟 IP（也必须与接口在同一网段，最多 16 个），不同的虚拟 IP 地址为不同的用户服务。
  - 实现多网关负载分担，需要重复执行上述 “主备备份” 的操作步骤，在接口上配置两个或多个 VRRP 备份组，各备份组之间以备份组号（virtual-router-id）区分。
- 设置设备在备份组中的优先级
  - 优先级越高，越会成为 master 设备。默认优先级是 100.
  - 优先级 0 为系统保留，优先级 255 保留给系统 IP 地址拥有者，即配置了路由器某个接口 IP 地址为虚拟路由器 IP 地址的设备，IP 地址拥有者的优先级不需要配置，也不可配置，直接为 255.
  - 如果优先级相同的情况下，vrrp 接口 IP 地址较大的接口所在设备为 master 设备。
  - 具体配置：

        [Huawei-GigabitEthernet0/0/1]vrrp vrid 2 priority 120
  - 缺省情况下，优先级值为100。