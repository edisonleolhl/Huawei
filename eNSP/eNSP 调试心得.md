# eNSP 调试心得

## 视图

- 用户视图：查看交换机的简单运行状态和统计信息

        <HUAWEI>
- 系统视图：配置系统参数

        [HUAWEI]
- 菜单视图：如vlan视图，路由视图，接口视图
- 从任意视图退回到用户视图：

        return
- 退回到上一视图：

        quit
- 命令行在线帮助
  - 完全帮助

        <HUAWEI>?
        <HUAWEI>display ?
  - 部分帮助

        <HUAWEI>d?
        <HUAWEI>display h?
- 配置相关命令：
  - 查看已保存的路由器

        display saved-configuration
  - 查看当前路由器配置

        display current-configuration
  - 保存当前配置

        save
  - 擦出存储设备中路由器配置文件

        reset saved-configuration
  - 比较配置文件

        compare configuration

## VLAN

### 基础知识

- 避免网络风暴
- 传统的以太网帧中插入4字节的 VLAN 标签，源/目的 MAC 地址之后
- VLAN 标签中的 VLAN ID 占12比特，所以共有2的12次方——4096个 VLAN 标签，一般不用首尾两个，所有一般能用的 VLAN 标签是4094个
- PVID（port default vlan id）：某交换机对 untagged 的标签会打上 PVID
- VLAN 划分方式
  - 基于接口划分（最简单）
  - 基于 MAC 地址划分
  - 基于 IP 子网地址划分
  - 基于协议划分
- VLAN 转发流程
![vlan](http://ooy7h5h7x.bkt.clouddn.com/vlan.jpg)
### 命令总结

- 查看 VLAN 信息：

        dis vlan 11

  会显示：

        [RT]dis vlan 11
        * : management-vlan
        ---------------------
        VLAN ID Type         Status   MAC Learning Broadcast/Multicast/Unicast Property 
        --------------------------------------------------------------------------------
        11      common       enable   enable       forward   forward   forward default  

        -------------------
        Untagged      Port: Ethernet0/0/0               Ethernet0/0/1               
                            Ethernet0/0/2               
        -------------------
        Active Untag  Port: Ethernet0/0/0               Ethernet0/0/1               
                            Ethernet0/0/2               
        ---------------------
        Interface                   Physical 
        Ethernet0/0/0               UP      
        Ethernet0/0/1               UP      
        Ethernet0/0/2               UP      

  可以看到：e 0/0/0 ~ e 0/0/2 加入了 VLAN 11          

- 修改端口类型时，报错:

        Error: Please renew the default configurations.

    解决方案：需要一层一层的来删除配置，直到恢复到默认的配置。

    比如：

    配置的时候为：

        port link-type access

        port default vlan 4

    如果直接更改端口模式就会报错为：

        Error: Please renew the default configurations.

    所以如果出现这种错误，在这里就需要从后往前删除, 即：

        undo port default vlan 4

        undo port link-type

    到这里以后 才可以重新更改端口的模式。

### 实例
- 基于接口划分 VLAN 配置示例（静态配置链路类型）
![alt](http://www.023wg.com/content/uploadfile/201601/41a41452930324.png)
  - 应用场景：
    - 某企业的交换机连接有很多用户，且相同业务用户通过不同的设备接入企业网络。
    - 为了通信的安全性，同时为了避免广播风暴，企业希望业务相同用户之间可以互相访问，业务不同用户不能直接访问。
    - 可以在交换机上配置基于接口划分 VLAN，把业务相同的用户连接的接口划分到同一 VLAN。这样属于不同 VLAN 的用户不能直接进行二层通信，同一 VLAN 内的用户可以直接互相通信。
  - 配置思路：
    - 创建 VLAN 并将连接用户的接口加入 VLAN，实现不同业务用户之间的二层流量隔离。
    - 配置 SwitchA 和 SwitchB 之间的链路类型及通过的 VLAN，实现相同业务用户通过 SwitchA 和 SwitchB 通信。
  - http://www.023wg.com/vlan/126.html
- 利用 VLANIF 实现不同 VLAN 之间的通信
![alt](http://www.023wg.com/content/uploadfile/201601/4f701453453169.png)
  - 应用场景：
    - 企业的不同用户拥有相同的业务，且位于不同的网段。
    - 现在相同业务的用户所属的 VLAN 不相同，需要实现不同 VLAN 中的用户相互通信。
  - 配置思路
    1. 创建 VLAN，确定用户所属的 VLAN。
    1. 配置接口加入 VLAN，允许用户所属的 VLAN 通过当前接口。
    1. 创建 VLANIF 接口并配置 IP 地址，实现三层互通。
    1. 为了成功实现 VLAN 间互通，VLAN 内主机的缺省网关必须是对应 VLANIF 接口的 IP 地址。
  - 检查配置结果
    - 在 VLAN10 中的 User1 主机上配置 IP 地址为 10.10.10.3/24，缺省网关为 VLANIF10 接口的 IP 地址 10.10.10.2/24。
    - 在 VLAN20 中的 User2 主机上配置 IP 地址为 10.10.20.3/24，缺省网关为 VLANIF20 接口的 IP 地址 10.10.20.2/24。
    - 配置完成后，VLAN10 内的 User1 与 VLAN20 内的 User2 能够相互访问。
  - http://www.023wg.com/vlan/138.html
- 