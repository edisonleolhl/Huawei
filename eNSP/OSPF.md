- router id 
  - CLI:

        router id 1.1.1.1
  - 一般与路由器的本地回环地址（loopback）相同
    - 因为本地回环地址相对于其他 IP 地址更加稳定的，不会宕掉，所以 router id 也比较稳定
  - 也可以不配置，OSPF 会自动配置（不推荐）
- OSPF 配置：

        OSPF 配置
        配置 RTA
        指定 Router ID
        [RTA]router id 1.1.1.1
        运行 OSPF
        [RTA]ospf
        创建区域 1
        [RTA-ospf-1]area 1
        在区域 1 视图下通告网络
        [RTA-ospf-1-area-0.0.0.1]network 10.1.1.0 0.0.0.3
        [RTA-ospf-1-area-0.0.0.1]network 1.1.1.1 0.0.0.0
  - 注意通告网络的配置，需要用反掩码
- loopback 接口
  - 可以在路由器包含的任意 area 中配置 loopback 接口
  - CLI：

        [RTA]int loopback 0
        [RTA-LoopBack0]ip add 1.1.1.1 255.255.255.255
- 查看 OSPF 信息
  - CLI：

       [RTA]display ospf brief

       [RTA]display ospf lsdb