<img src="https://cdn.nlark.com/yuque/0/2026/png/50508853/1776602078291-48c719d8-fd3c-4232-a3c8-cfd10ae767b4.png" width="717.3333333333334" title="" crop="0,0,1,1" id="ubcfa06d2" class="ne-image">

## <font style="color:rgb(15, 17, 21);">二层 VLAN + DHCP + AP 组网实验解析（结论先行）</font>
### <font style="color:rgb(15, 17, 21);">拓扑角色与核心需求推断</font>
+ **<font style="color:rgb(15, 17, 21);">LSW2</font>**<font style="color:rgb(15, 17, 21);">：二层交换机，配置 VLAN 和 STP（图中标注</font><font style="color:rgb(15, 17, 21);"> </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">STP DIS</font>`<font style="color:rgb(15, 17, 21);"> </font><font style="color:rgb(15, 17, 21);">表示可能</font>**<font style="color:rgb(15, 17, 21);">关闭生成树</font>**<font style="color:rgb(15, 17, 21);">），连接 DHCP Server、DNS、AP1、AP2 以及 Cloud2。</font>
+ **<font style="color:rgb(15, 17, 21);">DHCP Server</font>**<font style="color:rgb(15, 17, 21);">：为 AP1、AP2 以及其他终端分配 IP 地址。</font>
+ **<font style="color:rgb(15, 17, 21);">AP1 / AP2</font>**<font style="color:rgb(15, 17, 21);">：无线接入点，通过 DHCP 获取 IP 地址，并可能需要通过 Option 43 发现 AC（无线控制器）。</font>
+ **<font style="color:rgb(15, 17, 21);">Cloud2</font>**<font style="color:rgb(15, 17, 21);">：连接外部网络或管理主机。</font>
+ **<font style="color:rgb(15, 17, 21);">二层 VLAN</font>**<font style="color:rgb(15, 17, 21);">：用于隔离广播域，例如管理 VLAN、业务 VLAN 等。</font>

**<font style="color:rgb(15, 17, 21);">注意</font>**<font style="color:rgb(15, 17, 21);">：图中没有给出 AC（无线控制器），AP 可能工作在</font><font style="color:rgb(15, 17, 21);"> </font>**<font style="color:rgb(15, 17, 21);">胖 AP（FAT AP）</font>**<font style="color:rgb(15, 17, 21);"> </font><font style="color:rgb(15, 17, 21);">模式独立工作，或通过 Option 43 指向外部的 AC。</font>

---

## <font style="color:rgb(15, 17, 21);">一、VLAN 与二层交换配置</font>
### <font style="color:rgb(15, 17, 21);">1.1 VLAN 规划（示例）</font>
| <font style="color:rgb(15, 17, 21);">VLAN ID</font> | <font style="color:rgb(15, 17, 21);">用途</font> | <font style="color:rgb(15, 17, 21);">IP 网段（假设）</font> | <font style="color:rgb(15, 17, 21);">说明</font> |
| --- | --- | --- | --- |
| <font style="color:rgb(15, 17, 21);">10</font> | <font style="color:rgb(15, 17, 21);">管理 VLAN</font> | <font style="color:rgb(15, 17, 21);">172.16.1.0/24</font> | <font style="color:rgb(15, 17, 21);">用于 AP、交换机管理</font> |
| <font style="color:rgb(15, 17, 21);">20</font> | <font style="color:rgb(15, 17, 21);">业务 VLAN</font> | <font style="color:rgb(15, 17, 21);">192.168.10.0/24</font> | <font style="color:rgb(15, 17, 21);">无线用户接入</font> |
| <font style="color:rgb(15, 17, 21);">30</font> | <font style="color:rgb(15, 17, 21);">DHCP Server 专属</font> | <font style="color:rgb(15, 17, 21);">172.16.10.0/24</font> | <font style="color:rgb(15, 17, 21);">连接 DHCP 服务器</font> |


**<font style="color:rgb(15, 17, 21);">配置命令</font>**<font style="color:rgb(15, 17, 21);">（在 LSW2 上）：</font>

<font style="color:rgb(15, 17, 21);">bash</font>

```plain
[LSW2] vlan batch 10 20 30
[LSW2] interface GigabitEthernet0/0/1   # 连接 DHCP Server
[LSW2-GigabitEthernet0/0/1] port link-type access
[LSW2-GigabitEthernet0/0/1] port default vlan 30
[LSW2] interface GigabitEthernet0/0/2   # 连接 AP1
[LSW2-GigabitEthernet0/0/2] port link-type access
[LSW2-GigabitEthernet0/0/2] port default vlan 10   # 管理 VLAN
[LSW2] interface GigabitEthernet0/0/3   # 连接 AP2
[LSW2-GigabitEthernet0/0/3] port link-type access
[LSW2-GigabitEthernet0/0/3] port default vlan 10
[LSW2] interface GigabitEthernet0/0/4   # 连接 DNS（假设 DNS 在管理 VLAN）
[LSW2-GigabitEthernet0/0/4] port link-type access
[LSW2-GigabitEthernet0/0/4] port default vlan 10
[LSW2] interface GigabitEthernet0/0/5   # 连接 Cloud2（可能为 Trunk 或 Access）
[LSW2-GigabitEthernet0/0/5] port link-type trunk
[LSW2-GigabitEthernet0/0/5] port trunk allow-pass vlan 10 20 30
```

**<font style="color:rgb(15, 17, 21);">知识点</font>**<font style="color:rgb(15, 17, 21);">：</font>

+ <font style="color:rgb(15, 17, 21);">Access 端口连接终端设备（AP、服务器），Trunk 端口连接上游网络或云。</font>
+ <font style="color:rgb(15, 17, 21);">如果 AP 需要同时传输管理 VLAN 和业务 VLAN，应使用 Trunk 口（但图中 AP 仅接一根线，可能为 Access 管理 VLAN，业务 VLAN 通过 CAPWAP 隧道传递）。</font>

### <font style="color:rgb(15, 17, 21);">1.2 关闭 STP（</font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">STP DIS</font>`<font style="color:rgb(15, 17, 21);">）</font>
<font style="color:rgb(15, 17, 21);">bash</font>

<font style="color:rgb(15, 17, 21);">[LSW2] stp disable   # 全局关闭生成树</font>

**<font style="color:rgb(15, 17, 21);">为什么关闭？</font>**

+ <font style="color:rgb(15, 17, 21);">如果网络确实没有环路（树形拓扑），关闭 STP 可以避免端口状态迁移延迟。</font>
+ <font style="color:rgb(15, 17, 21);">实验环境或小规模网络常关闭 STP 简化配置。</font>
+ **<font style="color:rgb(15, 17, 21);">风险</font>**<font style="color:rgb(15, 17, 21);">：误接环路会导致广播风暴，需确保物理拓扑无环。</font>

---

## <font style="color:rgb(15, 17, 21);">二、DHCP 服务器配置（为 AP 分配 IP）</font>
### <font style="color:rgb(15, 17, 21);">2.1 假设 DHCP Server 连接在 VLAN 30，网段 172.16.10.0/24</font>
**<font style="color:rgb(15, 17, 21);">DHCP Server 上的配置</font>**<font style="color:rgb(15, 17, 21);">（以 Linux 或路由器为例，此处用华为路由器模拟）：</font>

<font style="color:rgb(15, 17, 21);">bash</font>

```plain
[DHCP_Server] dhcp enable
[DHCP_Server] interface GigabitEthernet0/0/0
[DHCP_Server-GigabitEthernet0/0/0] ip address 172.16.10.2 24
[DHCP_Server-GigabitEthernet0/0/0] dhcp select interface
[DHCP_Server-GigabitEthernet0/0/0] dhcp server dns-list 172.16.10.3   # DNS 服务器 IP
```

**<font style="color:rgb(15, 17, 21);">AP 获取 IP</font>**<font style="color:rgb(15, 17, 21);">：AP 启动后广播 DHCP Discover，LSW2 在 VLAN 10 内转发，DHCP Server 响应 Offer。</font>

### <font style="color:rgb(15, 17, 21);">2.2 为 AP 提供 AC 发现（Option 43）</font>
<font style="color:rgb(15, 17, 21);">如果 AP 需要注册到无线控制器（AC），DHCP 应下发 Option 43：</font>

<font style="color:rgb(15, 17, 21);">bash</font>

```plain
[DHCP_Server] ip pool ap-pool
[DHCP_Server-ip-pool-ap-pool] network 172.16.10.0 mask 24
[DHCP_Server-ip-pool-ap-pool] gateway-list 172.16.10.1
[DHCP_Server-ip-pool-ap-pool] option 43 sub-option 2 ip 192.168.100.1   # AC 的 IP
```

**<font style="color:rgb(15, 17, 21);">命令解析</font>**<font style="color:rgb(15, 17, 21);">：</font>

+ `<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">option 43 sub-option 2</font>`<font style="color:rgb(15, 17, 21);">：表示使用十进制 IP 列表格式（华为常用）。</font>
+ <font style="color:rgb(15, 17, 21);">AP 获取到 IP 后，会通过 Option 43 得知 AC 地址，发起 CAPWAP 隧道。</font>

---

## <font style="color:rgb(15, 17, 21);">三、AP（无线接入点）工作模式</font>
### <font style="color:rgb(15, 17, 21);">3.1 胖 AP（FAT AP）模式</font>
+ <font style="color:rgb(15, 17, 21);">AP 独立配置 SSID、加密、VLAN 等，无需 AC。</font>
+ <font style="color:rgb(15, 17, 21);">配置方式：Web 或命令行，直接指定业务 VLAN。</font>

### <font style="color:rgb(15, 17, 21);">3.2 瘦 AP（FIT AP）模式</font>
+ <font style="color:rgb(15, 17, 21);">AP 通过 DHCP 获取 IP，然后通过 Option 43 或广播发现 AC。</font>
+ <font style="color:rgb(15, 17, 21);">AC 集中管理配置，AP 仅转发数据。</font>

**<font style="color:rgb(15, 17, 21);">本图中没有 AC，可能 AP 为胖 AP</font>**<font style="color:rgb(15, 17, 21);">，或 AC 在 Cloud2 方向（通过上层网络）。</font>

---

## <font style="color:rgb(15, 17, 21);">四、DNS 服务器配置</font>
+ <font style="color:rgb(15, 17, 21);">DNS 服务器 IP 假设为</font><font style="color:rgb(15, 17, 21);"> </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">172.16.10.3</font>`<font style="color:rgb(15, 17, 21);">。</font>
+ <font style="color:rgb(15, 17, 21);">为 AP 提供域名解析（例如解析 AC 域名）。</font>

---

## <font style="color:rgb(15, 17, 21);">五、Cloud2 桥接（eNSP 云设备）</font>
+ <font style="color:rgb(15, 17, 21);">Cloud2 用于连接 eNSP 拓扑与物理主机或虚拟机。</font>
+ <font style="color:rgb(15, 17, 21);">配置要点：</font>
    1. <font style="color:rgb(15, 17, 21);">绑定物理网卡或虚拟环回网卡。</font>
    2. <font style="color:rgb(15, 17, 21);">创建端口映射，勾选“双向通道”。</font>
    3. <font style="color:rgb(15, 17, 21);">确保 LSW2 的</font><font style="color:rgb(15, 17, 21);"> </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">GE0/0/5</font>`<font style="color:rgb(15, 17, 21);"> </font><font style="color:rgb(15, 17, 21);">端口模式与 Cloud2 匹配（通常为 Trunk 或 Access）。</font>

**<font style="color:rgb(15, 17, 21);">示例配置</font>**<font style="color:rgb(15, 17, 21);">（在 eNSP Cloud2 中）：</font>

+ <font style="color:rgb(15, 17, 21);">绑定信息：选择</font><font style="color:rgb(15, 17, 21);"> </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">UDP</font>`<font style="color:rgb(15, 17, 21);">，绑定</font><font style="color:rgb(15, 17, 21);"> </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">Microsoft KM-TEST Loopback Adapter</font>`<font style="color:rgb(15, 17, 21);">。</font>
+ <font style="color:rgb(15, 17, 21);">端口映射：入端口编号 5，出端口编号 5，勾选双向通道。</font>

---

## <font style="color:rgb(15, 17, 21);">六、验证命令</font>
| <font style="color:rgb(15, 17, 21);">设备</font> | <font style="color:rgb(15, 17, 21);">命令</font> | <font style="color:rgb(15, 17, 21);">预期结果</font> |
| --- | --- | --- |
| <font style="color:rgb(15, 17, 21);">LSW2</font> | `<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">display vlan</font>` | <font style="color:rgb(15, 17, 21);">端口正确划分 VLAN</font> |
| <font style="color:rgb(15, 17, 21);">LSW2</font> | `<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">display stp</font>` | <font style="color:rgb(15, 17, 21);">STP 状态为</font><font style="color:rgb(15, 17, 21);"> </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">Disabled</font>` |
| <font style="color:rgb(15, 17, 21);">DHCP Server</font> | `<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">display dhcp server lease</font>` | <font style="color:rgb(15, 17, 21);">有 AP 的 IP 分配记录</font> |
| <font style="color:rgb(15, 17, 21);">AP（通过管理界面）</font> | `<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">show ip interface brief</font>` | <font style="color:rgb(15, 17, 21);">获取到 172.16.10.x IP</font> |
| <font style="color:rgb(15, 17, 21);">AP</font> | `<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">ping 172.16.10.2</font>` | <font style="color:rgb(15, 17, 21);">通（DHCP 服务器）</font> |
| <font style="color:rgb(15, 17, 21);">外网主机（通过 Cloud2）</font> | `<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">ping 172.16.10.100</font>`<br/><font style="color:rgb(15, 17, 21);">（AP IP）</font> | <font style="color:rgb(15, 17, 21);">通（需路由）</font> |


---

## <font style="color:rgb(15, 17, 21);">七、常见问题与排错</font>
| <font style="color:rgb(15, 17, 21);">现象</font> | <font style="color:rgb(15, 17, 21);">可能原因</font> | <font style="color:rgb(15, 17, 21);">解决方法</font> |
| --- | --- | --- |
| <font style="color:rgb(15, 17, 21);">AP 获取不到 IP</font> | <font style="color:rgb(15, 17, 21);">DHCP 中继未配置（跨 VLAN）或服务器未开启</font> | <font style="color:rgb(15, 17, 21);">确认 AP 所在 VLAN 的网关设备配置了</font><font style="color:rgb(15, 17, 21);"> </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">dhcp relay</font>` |
| <font style="color:rgb(15, 17, 21);">AP 无法发现 AC</font> | <font style="color:rgb(15, 17, 21);">Option 43 未配置或 AC IP 错误</font> | <font style="color:rgb(15, 17, 21);">检查 DHCP 地址池中的</font><font style="color:rgb(15, 17, 21);"> </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">option 43</font>`<br/><font style="color:rgb(15, 17, 21);"> </font><font style="color:rgb(15, 17, 21);">配置</font> |
| <font style="color:rgb(15, 17, 21);">Cloud2 不通</font> | <font style="color:rgb(15, 17, 21);">绑定错误或防火墙拦截</font> | <font style="color:rgb(15, 17, 21);">检查 Cloud 端口映射，关闭 Windows 防火墙</font> |
| <font style="color:rgb(15, 17, 21);">VLAN 间不通</font> | <font style="color:rgb(15, 17, 21);">三层路由缺失</font> | <font style="color:rgb(15, 17, 21);">如需跨 VLAN 通信，需配置 VLANIF 接口或路由器</font> |


---

## <font style="color:rgb(15, 17, 21);">八、知识点拓展</font>
+ **<font style="color:rgb(15, 17, 21);">STP 关闭的风险与适用场景</font>**<font style="color:rgb(15, 17, 21);">：在无环路的树形拓扑中可关闭，但必须谨慎。</font>
+ **<font style="color:rgb(15, 17, 21);">Option 43 的多种格式</font>**<font style="color:rgb(15, 17, 21);">：</font>
    - <font style="color:rgb(15, 17, 21);">子选项 2：十进制 IP 列表（</font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">ip 192.168.1.1</font>`<font style="color:rgb(15, 17, 21);">）</font>
    - <font style="color:rgb(15, 17, 21);">子选项 1：十六进制（</font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">hex 80070000c0a80101</font>`<font style="color:rgb(15, 17, 21);">）</font>
+ **<font style="color:rgb(15, 17, 21);">AP 注册过程（CAPWAP）</font>**<font style="color:rgb(15, 17, 21);">：</font>
    1. <font style="color:rgb(15, 17, 21);">AP 获取 IP。</font>
    2. <font style="color:rgb(15, 17, 21);">通过广播、Option 43 或 DNS 发现 AC。</font>
    3. <font style="color:rgb(15, 17, 21);">建立 DTLS 隧道（可选加密）。</font>
    4. <font style="color:rgb(15, 17, 21);">下载配置并发射 SSID。</font>

---

## <font style="color:rgb(15, 17, 21);">九、总结表</font>
| <font style="color:rgb(15, 17, 21);">组件</font> | <font style="color:rgb(15, 17, 21);">关键配置</font> | <font style="color:rgb(15, 17, 21);">验证命令</font> |
| --- | --- | --- |
| <font style="color:rgb(15, 17, 21);">LSW2 VLAN</font> | `<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">vlan batch</font>`<br/><font style="color:rgb(15, 17, 21);">,</font><font style="color:rgb(15, 17, 21);"> </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">port default vlan</font>` | `<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">display vlan</font>` |
| <font style="color:rgb(15, 17, 21);">LSW2 STP 关闭</font> | `<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">stp disable</font>` | `<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">display stp</font>` |
| <font style="color:rgb(15, 17, 21);">DHCP Server</font> | `<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">ip pool</font>`<br/><font style="color:rgb(15, 17, 21);">,</font><font style="color:rgb(15, 17, 21);"> </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">option 43</font>` | `<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">display dhcp server lease</font>` |
| <font style="color:rgb(15, 17, 21);">AP</font> | <font style="color:rgb(15, 17, 21);">DHCP 获取 IP，Option 43 解析</font> | `<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">display ip interface brief</font>` |
| <font style="color:rgb(15, 17, 21);">Cloud2</font> | <font style="color:rgb(15, 17, 21);">绑定网卡，双向通道</font> | <font style="color:rgb(15, 17, 21);">物理主机 ping 模拟器内 IP</font> |

