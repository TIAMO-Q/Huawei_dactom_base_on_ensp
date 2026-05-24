
配置主机名和 IP 地址
<Huawei> system-view
[Huawei] sysname R1
[R1] interface GigabitEthernet0/0/0
[R1-GigabitEthernet0/0/0] ip address 192.168.1.1 24
[R1-GigabitEthernet0/0/0] quit
配置 R1 为 Password 认证，级别=15
[R1] user-interface vty 0 4
[R1-ui-vty0-4] authentication-mode password
[R1-ui-vty0-4] set authentication password cipher Huawei@123
[R1-ui-vty0-4] user privilege level 15
[R1-ui-vty0-4] quit
● vty 0 4 表示虚拟终端线路 0~4（共5个会话）。
● authentication-mode password 使用密码认证。
● set authentication password cipher 设置密码（密文存储）。
● user privilege level 15 设置用户级别为 15（最高权限，相当于管理员）
配置 R2 为 AAA 认证，用户名为 wakin，级别=1
[R2] aaa
[R2-aaa] local-user wakin password cipher wakin@123
[R2-aaa] local-user wakin privilege level 2
[R2-aaa] local-user wakin service-type telnet
[R2-aaa] quit
[R2] user-interface vty 0 4 
[R2-ui-vty0-4] authentication-mode aaa
[R2-ui-vty0-4] quit
● aaa 进入 AAA 视图。
● local-user 创建本地用户 wakin，密码密文。
● privilege level 1 设置用户级别为 1（最低权限，只能执行少量查看命令，如 ping、display 等）。
● service-type telnet 允许该用户通过 Telnet 登录。
● user-interface vty 下配置 authentication-mode aaa，表示使用 AAA 认证（即本地用户数据库）。
配置 R3 为 AAA 认证，用户名为 winnie，级别=2
[R3] aaa
[R3-aaa] local-user winnie password cipher winnie@123
[R3-aaa] local-user winnie privilege level 2
[R3-aaa] local-user winnie service-type telnet
[R3-aaa] quit
[R3] user-interface vty 0 4
[R3-ui-vty0-4] authentication-mode aaa
[R3-ui-vty0-4] quit
一、配置命令（以华为设备为例）
拓扑假设
● 三台路由器通过以太网口互联，IP 地址规划如下（可自定）：
  ○ R1: GE0/0/0 IP 192.168.1.1/24
  ○ R2: GE0/0/0 IP 192.168.1.2/24
  ○ R3: GE0/0/0 IP 192.168.1.3/24
● 所有接口在同一广播域，可互相 ping 通。
1. 配置主机名和 IP 地址
R1
bash
<Huawei> system-view
[Huawei] sysname R1
[R1] interface GigabitEthernet0/0/0
[R1-GigabitEthernet0/0/0] ip address 192.168.1.1 24
[R1-GigabitEthernet0/0/0] quit
R2
bash
<Huawei> system-view
[Huawei] sysname R2
[R2] interface GigabitEthernet0/0/0
[R2-GigabitEthernet0/0/0] ip address 192.168.1.2 24
[R2-GigabitEthernet0/0/0] quit
R3
bash
<Huawei> system-view
[Huawei] sysname R3
[R3] interface GigabitEthernet0/0/0
[R3-GigabitEthernet0/0/0] ip address 192.168.1.3 24
[R3-GigabitEthernet0/0/0] quit
2. 配置 R1 为 Password 认证，级别=15
bash
[R1] user-interface vty 0 4
[R1-ui-vty0-4] authentication-mode password
[R1-ui-vty0-4] set authentication password cipher Huawei@123
[R1-ui-vty0-4] user privilege level 15
[R1-ui-vty0-4] quit
说明：
● vty 0 4 表示虚拟终端线路 0~4（共5个会话）。
● authentication-mode password 使用密码认证。
● set authentication password cipher 设置密码（密文存储）。
● user privilege level 15 设置用户级别为 15（最高权限，相当于管理员）。
3. 配置 R2 为 AAA 认证，用户名为 wakin，级别=1
bash
[R2] aaa
[R2-aaa] local-user wakin password cipher wakin@123
[R2-aaa] local-user wakin privilege level 1
[R2-aaa] local-user wakin service-type telnet
[R2-aaa] quit
[R2] user-interface vty 0 4
[R2-ui-vty0-4] authentication-mode aaa
[R2-ui-vty0-4] quit
说明：
● aaa 进入 AAA 视图。
● local-user 创建本地用户 wakin，密码密文。
● privilege level 1 设置用户级别为 1（最低权限，只能执行少量查看命令，如 ping、display 等）。
● service-type telnet 允许该用户通过 Telnet 登录。
● user-interface vty 下配置 authentication-mode aaa，表示使用 AAA 认证（即本地用户数据库）。
4. 配置 R3 为 AAA 认证，用户名为 winnie，级别=2
bash
[R3] aaa
[R3-aaa] local-user winnie password cipher winnie@123
[R3-aaa] local-user winnie privilege level 2
[R3-aaa] local-user winnie service-type telnet
[R3-aaa] quit
[R3] user-interface vty 0 4
[R3-ui-vty0-4] authentication-mode aaa
[R3-ui-vty0-4] quit
说明：
● 级别 2 比级别 1 权限稍高，通常可以执行部分系统配置命令，但仍有限制。

二、相关知识点总结
1. 路由器远程登录认证方式
● Password 认证：所有用户使用同一个密码登录，配置简单，但安全性低，无法区分不同用户权限。
● AAA 认证：使用本地用户数据库或外部服务器（如 RADIUS）进行认证，可设置不同用户名、密码和权限级别，安全性高。
2. 用户权限级别（华为 VRP）
级别	名称	说明
0	访问级	只能执行少量网络诊断命令（如 ping
、tracert
）。
1	监控级	可执行 display
 等查看命令，不能修改配置。
2	配置级	可执行部分系统配置命令，但无法修改关键安全配置。
3-14	管理级	中间级别，通常用于自定义权限。
15	超级管理级	拥有全部权限（等同于 root
）。
3. VTY（Virtual Type Terminal）线路
● VTY 是虚拟终端线路，用于远程登录（Telnet、SSH）。
● 通常配置 user-interface vty 0 4 表示 0~4 共 5 条线路，可同时支持 5 个远程会话。
● 每条 VTY 线路可独立配置认证方式和权限，但通常统一配置。
4. AAA 模型
● Authentication（认证）：验证用户身份（用户名/密码）。
● Authorization（授权）：确定用户能执行哪些命令（通过权限级别或自定义策略）。
● Accounting（计费）：记录用户操作日志（可选）。
5. 常见排错命令
● display users：查看当前登录用户及线路。
● display user-interface：查看 VTY 线路配置状态。
● display aaa local-user：查看本地用户配置。
● telnet <ip> 测试登录，观察是否提示密码或用户名。
6. 安全建议
● 生产环境中推荐使用 SSH 代替 Telnet（Telnet 明文传输）。
● 使用 AAA 认证并为不同用户分配最低必要权限。
● 定期修改密码，并使用 cipher 参数存储密文。


