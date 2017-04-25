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
