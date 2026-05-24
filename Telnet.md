<img src="https://cdn.nlark.com/yuque/0/2026/png/50508853/1776425927855-d2abcb75-2282-4142-9bcb-8f45149cf392.png" width="841.3333333333334" title="" crop="0,0,1,1" id="uc8fe4015" class="ne-image">

### <font style="color:rgb(15, 17, 21);">配置主机名和 IP 地址</font>
<Huawei> system-view

[Huawei] sysname R1

[R1] interface GigabitEthernet0/0/0

[R1-GigabitEthernet0/0/0] ip address 192.168.1.1 24

[R1-GigabitEthernet0/0/0] quit

### <font style="color:rgb(15, 17, 21);">配置 R1 为 Password 认证，级别=15</font>
[R1] user-interface vty 0 4

[R1-ui-vty0-4] authentication-mode password

[R1-ui-vty0-4] set authentication password cipher Huawei@123

[R1-ui-vty0-4] user privilege level 15

[R1-ui-vty0-4] quit

+ `<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">vty 0 4</font>`<font style="color:rgb(15, 17, 21);"> </font><font style="color:rgb(15, 17, 21);">表示虚拟终端线路 0~4（共5个会话）。</font>
+ `<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">authentication-mode password</font>`<font style="color:rgb(15, 17, 21);"> </font><font style="color:rgb(15, 17, 21);">使用密码认证。</font>
+ `<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">set authentication password cipher</font>`<font style="color:rgb(15, 17, 21);"> </font><font style="color:rgb(15, 17, 21);">设置密码（密文存储）。</font>
+ `<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">user privilege level 15</font>`<font style="color:rgb(15, 17, 21);"> 设置用户级别为 15（最高权限，相当于管理员）</font>

### <font style="color:rgb(15, 17, 21);">配置 R2 为 AAA 认证，用户名为 wakin，级别=1</font>
[R2] aaa

[R2-aaa] local-user wakin password cipher wakin@123

[R2-aaa] local-user wakin privilege level 2

[R2-aaa] local-user wakin service-type telnet

[R2-aaa] quit

[R2] user-interface vty 0 4 

[R2-ui-vty0-4] authentication-mode aaa

[R2-ui-vty0-4] quit

+ `<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">aaa</font>`<font style="color:rgb(15, 17, 21);"> </font><font style="color:rgb(15, 17, 21);">进入 AAA 视图。</font>
+ `<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">local-user</font>`<font style="color:rgb(15, 17, 21);"> </font><font style="color:rgb(15, 17, 21);">创建本地用户</font><font style="color:rgb(15, 17, 21);"> </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">wakin</font>`<font style="color:rgb(15, 17, 21);">，密码密文。</font>
+ `<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">privilege level 1</font>`<font style="color:rgb(15, 17, 21);"> </font><font style="color:rgb(15, 17, 21);">设置用户级别为 1（最低权限，只能执行少量查看命令，如</font><font style="color:rgb(15, 17, 21);"> </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">ping</font>`<font style="color:rgb(15, 17, 21);">、</font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">display</font>`<font style="color:rgb(15, 17, 21);"> </font><font style="color:rgb(15, 17, 21);">等）。</font>
+ `<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">service-type telnet</font>`<font style="color:rgb(15, 17, 21);"> </font><font style="color:rgb(15, 17, 21);">允许该用户通过 Telnet 登录。</font>
+ `<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">user-interface vty</font>`<font style="color:rgb(15, 17, 21);"> 下配置 </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">authentication-mode aaa</font>`<font style="color:rgb(15, 17, 21);">，表示使用 AAA 认证（即本地用户数据库）。</font>

### <font style="color:rgb(15, 17, 21);">配置 R3 为 AAA 认证，用户名为 winnie，级别=2</font>
[R3] aaa

[R3-aaa] local-user winnie password cipher winnie@123

[R3-aaa] local-user winnie privilege level 2

[R3-aaa] local-user winnie service-type telnet

[R3-aaa] quit

[R3] user-interface vty 0 4

[R3-ui-vty0-4] authentication-mode aaa

[R3-ui-vty0-4] quit

## <font style="color:rgb(15, 17, 21);">一、配置命令（以华为设备为例）</font>
### <font style="color:rgb(15, 17, 21);">拓扑假设</font>
+ <font style="color:rgb(15, 17, 21);">三台路由器通过以太网口互联，IP 地址规划如下（可自定）：</font>
    - <font style="color:rgb(15, 17, 21);">R1: GE0/0/0 IP 192.168.1.1/24</font>
    - <font style="color:rgb(15, 17, 21);">R2: GE0/0/0 IP 192.168.1.2/24</font>
    - <font style="color:rgb(15, 17, 21);">R3: GE0/0/0 IP 192.168.1.3/24</font>
+ <font style="color:rgb(15, 17, 21);">所有接口在同一广播域，可互相 ping 通。</font>

### <font style="color:rgb(15, 17, 21);">1. 配置主机名和 IP 地址</font>
#### <font style="color:rgb(15, 17, 21);">R1</font>
<font style="color:rgb(15, 17, 21);">bash</font>

```plain
<Huawei> system-view
[Huawei] sysname R1
[R1] interface GigabitEthernet0/0/0
[R1-GigabitEthernet0/0/0] ip address 192.168.1.1 24
[R1-GigabitEthernet0/0/0] quit
```

#### <font style="color:rgb(15, 17, 21);">R2</font>
<font style="color:rgb(15, 17, 21);">bash</font>

```plain
<Huawei> system-view
[Huawei] sysname R2
[R2] interface GigabitEthernet0/0/0
[R2-GigabitEthernet0/0/0] ip address 192.168.1.2 24
[R2-GigabitEthernet0/0/0] quit
```

#### <font style="color:rgb(15, 17, 21);">R3</font>
<font style="color:rgb(15, 17, 21);">bash</font>

```plain
<Huawei> system-view
[Huawei] sysname R3
[R3] interface GigabitEthernet0/0/0
[R3-GigabitEthernet0/0/0] ip address 192.168.1.3 24
[R3-GigabitEthernet0/0/0] quit
```

### <font style="color:rgb(15, 17, 21);">2. 配置 R1 为 Password 认证，级别=15</font>
<font style="color:rgb(15, 17, 21);">bash</font>

```plain
[R1] user-interface vty 0 4
[R1-ui-vty0-4] authentication-mode password
[R1-ui-vty0-4] set authentication password cipher Huawei@123
[R1-ui-vty0-4] user privilege level 15
[R1-ui-vty0-4] quit
```

**<font style="color:rgb(15, 17, 21);">说明</font>**<font style="color:rgb(15, 17, 21);">：</font>

+ `<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">vty 0 4</font>`<font style="color:rgb(15, 17, 21);"> </font><font style="color:rgb(15, 17, 21);">表示虚拟终端线路 0~4（共5个会话）。</font>
+ `<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">authentication-mode password</font>`<font style="color:rgb(15, 17, 21);"> </font><font style="color:rgb(15, 17, 21);">使用密码认证。</font>
+ `<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">set authentication password cipher</font>`<font style="color:rgb(15, 17, 21);"> </font><font style="color:rgb(15, 17, 21);">设置密码（密文存储）。</font>
+ `<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">user privilege level 15</font>`<font style="color:rgb(15, 17, 21);"> </font><font style="color:rgb(15, 17, 21);">设置用户级别为 15（最高权限，相当于管理员）。</font>

### <font style="color:rgb(15, 17, 21);">3. 配置 R2 为 AAA 认证，用户名为 wakin，级别=1</font>
<font style="color:rgb(15, 17, 21);">bash</font>

```plain
[R2] aaa
[R2-aaa] local-user wakin password cipher wakin@123
[R2-aaa] local-user wakin privilege level 1
[R2-aaa] local-user wakin service-type telnet
[R2-aaa] quit
[R2] user-interface vty 0 4
[R2-ui-vty0-4] authentication-mode aaa
[R2-ui-vty0-4] quit
```

**<font style="color:rgb(15, 17, 21);">说明</font>**<font style="color:rgb(15, 17, 21);">：</font>

+ `<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">aaa</font>`<font style="color:rgb(15, 17, 21);"> </font><font style="color:rgb(15, 17, 21);">进入 AAA 视图。</font>
+ `<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">local-user</font>`<font style="color:rgb(15, 17, 21);"> </font><font style="color:rgb(15, 17, 21);">创建本地用户</font><font style="color:rgb(15, 17, 21);"> </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">wakin</font>`<font style="color:rgb(15, 17, 21);">，密码密文。</font>
+ `<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">privilege level 1</font>`<font style="color:rgb(15, 17, 21);"> </font><font style="color:rgb(15, 17, 21);">设置用户级别为 1（最低权限，只能执行少量查看命令，如</font><font style="color:rgb(15, 17, 21);"> </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">ping</font>`<font style="color:rgb(15, 17, 21);">、</font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">display</font>`<font style="color:rgb(15, 17, 21);"> </font><font style="color:rgb(15, 17, 21);">等）。</font>
+ `<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">service-type telnet</font>`<font style="color:rgb(15, 17, 21);"> </font><font style="color:rgb(15, 17, 21);">允许该用户通过 Telnet 登录。</font>
+ `<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">user-interface vty</font>`<font style="color:rgb(15, 17, 21);"> </font><font style="color:rgb(15, 17, 21);">下配置</font><font style="color:rgb(15, 17, 21);"> </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">authentication-mode aaa</font>`<font style="color:rgb(15, 17, 21);">，表示使用 AAA 认证（即本地用户数据库）。</font>

### <font style="color:rgb(15, 17, 21);">4. 配置 R3 为 AAA 认证，用户名为 winnie，级别=2</font>
<font style="color:rgb(15, 17, 21);">bash</font>

```plain
[R3] aaa
[R3-aaa] local-user winnie password cipher winnie@123
[R3-aaa] local-user winnie privilege level 2
[R3-aaa] local-user winnie service-type telnet
[R3-aaa] quit
[R3] user-interface vty 0 4
[R3-ui-vty0-4] authentication-mode aaa
[R3-ui-vty0-4] quit
```

**<font style="color:rgb(15, 17, 21);">说明</font>**<font style="color:rgb(15, 17, 21);">：</font>

+ <font style="color:rgb(15, 17, 21);">级别 2 比级别 1 权限稍高，通常可以执行部分系统配置命令，但仍有限制。</font>

---

## <font style="color:rgb(15, 17, 21);">二、相关知识点总结</font>
### <font style="color:rgb(15, 17, 21);">1. 路由器远程登录认证方式</font>
+ **<font style="color:rgb(15, 17, 21);">Password 认证</font>**<font style="color:rgb(15, 17, 21);">：所有用户使用同一个密码登录，配置简单，但安全性低，无法区分不同用户权限。</font>
+ **<font style="color:rgb(15, 17, 21);">AAA 认证</font>**<font style="color:rgb(15, 17, 21);">：使用本地用户数据库或外部服务器（如 RADIUS）进行认证，可设置不同用户名、密码和权限级别，安全性高。</font>

### <font style="color:rgb(15, 17, 21);">2. 用户权限级别（华为 VRP）</font>
| <font style="color:rgb(15, 17, 21);">级别</font> | <font style="color:rgb(15, 17, 21);">名称</font> | <font style="color:rgb(15, 17, 21);">说明</font> |
| --- | --- | --- |
| <font style="color:rgb(15, 17, 21);">0</font> | <font style="color:rgb(15, 17, 21);">访问级</font> | <font style="color:rgb(15, 17, 21);">只能执行少量网络诊断命令（如</font><font style="color:rgb(15, 17, 21);"> </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">ping</font>`<br/><font style="color:rgb(15, 17, 21);">、</font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">tracert</font>`<br/><font style="color:rgb(15, 17, 21);">）。</font> |
| <font style="color:rgb(15, 17, 21);">1</font> | <font style="color:rgb(15, 17, 21);">监控级</font> | <font style="color:rgb(15, 17, 21);">可执行</font><font style="color:rgb(15, 17, 21);"> </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">display</font>`<br/><font style="color:rgb(15, 17, 21);"> </font><font style="color:rgb(15, 17, 21);">等查看命令，不能修改配置。</font> |
| <font style="color:rgb(15, 17, 21);">2</font> | <font style="color:rgb(15, 17, 21);">配置级</font> | <font style="color:rgb(15, 17, 21);">可执行部分系统配置命令，但无法修改关键安全配置。</font> |
| <font style="color:rgb(15, 17, 21);">3-14</font> | <font style="color:rgb(15, 17, 21);">管理级</font> | <font style="color:rgb(15, 17, 21);">中间级别，通常用于自定义权限。</font> |
| <font style="color:rgb(15, 17, 21);">15</font> | <font style="color:rgb(15, 17, 21);">超级管理级</font> | <font style="color:rgb(15, 17, 21);">拥有全部权限（等同于</font><font style="color:rgb(15, 17, 21);"> </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">root</font>`<br/><font style="color:rgb(15, 17, 21);">）。</font> |


### <font style="color:rgb(15, 17, 21);">3. VTY（Virtual Type Terminal）线路</font>
+ <font style="color:rgb(15, 17, 21);">VTY 是虚拟终端线路，用于远程登录（Telnet、SSH）。</font>
+ <font style="color:rgb(15, 17, 21);">通常配置</font><font style="color:rgb(15, 17, 21);"> </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">user-interface vty 0 4</font>`<font style="color:rgb(15, 17, 21);"> </font><font style="color:rgb(15, 17, 21);">表示 0~4 共 5 条线路，可同时支持 5 个远程会话。</font>
+ <font style="color:rgb(15, 17, 21);">每条 VTY 线路可独立配置认证方式和权限，但通常统一配置。</font>

### <font style="color:rgb(15, 17, 21);">4. AAA 模型</font>
+ **<font style="color:rgb(15, 17, 21);">Authentication（认证）</font>**<font style="color:rgb(15, 17, 21);">：验证用户身份（用户名/密码）。</font>
+ **<font style="color:rgb(15, 17, 21);">Authorization（授权）</font>**<font style="color:rgb(15, 17, 21);">：确定用户能执行哪些命令（通过权限级别或自定义策略）。</font>
+ **<font style="color:rgb(15, 17, 21);">Accounting（计费）</font>**<font style="color:rgb(15, 17, 21);">：记录用户操作日志（可选）。</font>

### <font style="color:rgb(15, 17, 21);">5. 常见排错命令</font>
+ `<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">display users</font>`<font style="color:rgb(15, 17, 21);">：查看当前登录用户及线路。</font>
+ `<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">display user-interface</font>`<font style="color:rgb(15, 17, 21);">：查看 VTY 线路配置状态。</font>
+ `<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">display aaa local-user</font>`<font style="color:rgb(15, 17, 21);">：查看本地用户配置。</font>
+ `<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">telnet <ip></font>`<font style="color:rgb(15, 17, 21);"> </font><font style="color:rgb(15, 17, 21);">测试登录，观察是否提示密码或用户名。</font>

### <font style="color:rgb(15, 17, 21);">6. 安全建议</font>
+ <font style="color:rgb(15, 17, 21);">生产环境中推荐使用</font><font style="color:rgb(15, 17, 21);"> </font>**<font style="color:rgb(15, 17, 21);">SSH</font>**<font style="color:rgb(15, 17, 21);"> </font><font style="color:rgb(15, 17, 21);">代替 Telnet（Telnet 明文传输）。</font>
+ <font style="color:rgb(15, 17, 21);">使用 AAA 认证并为不同用户分配最低必要权限。</font>
+ <font style="color:rgb(15, 17, 21);">定期修改密码，并使用 </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">cipher</font>`<font style="color:rgb(15, 17, 21);"> 参数存储密文。</font>

<font style="color:rgb(129, 133, 140);">  
</font>
