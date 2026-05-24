<img src="https://cdn.nlark.com/yuque/0/2026/png/50508853/1776568860954-0c48d290-a9f5-40c0-9ede-4941cad2319b.png" width="1130" title="" crop="0,0,1,1" id="u11500c87" class="ne-image">

## <font style="color:rgb(15, 17, 21);">一、整体拓扑与角色梳理</font>
+ **<font style="color:rgb(15, 17, 21);">R1</font>**<font style="color:rgb(15, 17, 21);">：作为</font><font style="color:rgb(15, 17, 21);"> </font>**<font style="color:rgb(15, 17, 21);">DHCP 服务器</font>**<font style="color:rgb(15, 17, 21);"> </font><font style="color:rgb(15, 17, 21);">和</font><font style="color:rgb(15, 17, 21);"> </font>**<font style="color:rgb(15, 17, 21);">OSPF 路由器</font>**<font style="color:rgb(15, 17, 21);">，连接两个直连网段（RoomA</font><font style="color:rgb(15, 17, 21);"> </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">172.16.10.0/24</font>`<font style="color:rgb(15, 17, 21);">、RoomB</font><font style="color:rgb(15, 17, 21);"> </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">172.16.20.0/24</font>`<font style="color:rgb(15, 17, 21);">）以及核心链路</font><font style="color:rgb(15, 17, 21);"> </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">13.0.0.0/24</font>`<font style="color:rgb(15, 17, 21);">。</font>
+ **<font style="color:rgb(15, 17, 21);">SW3</font>**<font style="color:rgb(15, 17, 21);">：三层交换机，创建了多个 VLAN（10,20,30,40,50）对应不同的房间（RoomC~G）。它作为</font><font style="color:rgb(15, 17, 21);"> </font>**<font style="color:rgb(15, 17, 21);">DHCP 中继（DHCP Relay）</font>**<font style="color:rgb(15, 17, 21);">，将来自这些 VLAN 的 DHCP 请求转发给 R1。同时它也运行 OSPF，与 R1 交换路由。</font>
+ **<font style="color:rgb(15, 17, 21);">PC1~PC5</font>**<font style="color:rgb(15, 17, 21);"> </font><font style="color:rgb(15, 17, 21);">等分别属于不同 VLAN，通过 SW3 的 VLANIF 接口作为网关，并通过 DHCP 中继获取 IP 地址。</font>

---

## <font style="color:rgb(15, 17, 21);">二、DHCP 原理与中继详解</font>
### <font style="color:rgb(15, 17, 21);">1. DHCP 基本交互（DORA）</font>
<font style="color:rgb(15, 17, 21);">DHCP（Dynamic Host Configuration Protocol）使用</font><font style="color:rgb(15, 17, 21);"> </font>**<font style="color:rgb(15, 17, 21);">UDP</font>**<font style="color:rgb(15, 17, 21);"> </font><font style="color:rgb(15, 17, 21);">端口：客户端</font><font style="color:rgb(15, 17, 21);"> </font>**<font style="color:rgb(15, 17, 21);">68</font>**<font style="color:rgb(15, 17, 21);">，服务器</font><font style="color:rgb(15, 17, 21);"> </font>**<font style="color:rgb(15, 17, 21);">67</font>**<font style="color:rgb(15, 17, 21);">。标准交互过程：</font>

| <font style="color:rgb(15, 17, 21);">步骤</font> | <font style="color:rgb(15, 17, 21);">报文</font> | <font style="color:rgb(15, 17, 21);">方向</font> | <font style="color:rgb(15, 17, 21);">关键内容</font> |
| --- | --- | --- | --- |
| <font style="color:rgb(15, 17, 21);">1</font> | **<font style="color:rgb(15, 17, 21);">Discover</font>** | <font style="color:rgb(15, 17, 21);">客户端 → 广播 (255.255.255.255)</font> | <font style="color:rgb(15, 17, 21);">客户端寻找 DHCP 服务器，源IP 0.0.0.0</font> |
| <font style="color:rgb(15, 17, 21);">2</font> | **<font style="color:rgb(15, 17, 21);">Offer</font>** | <font style="color:rgb(15, 17, 21);">服务器 → 广播/单播</font> | <font style="color:rgb(15, 17, 21);">服务器提供可用的 IP 地址及其他参数</font> |
| <font style="color:rgb(15, 17, 21);">3</font> | **<font style="color:rgb(15, 17, 21);">Request</font>** | <font style="color:rgb(15, 17, 21);">客户端 → 广播</font> | <font style="color:rgb(15, 17, 21);">客户端正式请求该 IP（同时告知其他服务器已选择）</font> |
| <font style="color:rgb(15, 17, 21);">4</font> | **<font style="color:rgb(15, 17, 21);">Ack</font>** | <font style="color:rgb(15, 17, 21);">服务器 → 广播/单播</font> | <font style="color:rgb(15, 17, 21);">服务器确认租约，客户端开始使用 IP</font> |


**<font style="color:rgb(15, 17, 21);">抓包细节</font>**<font style="color:rgb(15, 17, 21);">：在客户端接口抓包，过滤</font><font style="color:rgb(15, 17, 21);"> </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">bootp</font>`<font style="color:rgb(15, 17, 21);"> </font><font style="color:rgb(15, 17, 21);">或</font><font style="color:rgb(15, 17, 21);"> </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">udp.port==67</font>`<font style="color:rgb(15, 17, 21);">。你会看到 Discover 的目标 MAC 是全 F，源 MAC 是客户端。Offer 中会包含</font><font style="color:rgb(15, 17, 21);"> </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">Your (client) IP address</font>`<font style="color:rgb(15, 17, 21);">、</font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">Subnet Mask</font>`<font style="color:rgb(15, 17, 21);">、</font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">Router</font>`<font style="color:rgb(15, 17, 21);">（网关）、</font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">DNS</font>`<font style="color:rgb(15, 17, 21);"> </font><font style="color:rgb(15, 17, 21);">等选项。</font>

### <font style="color:rgb(15, 17, 21);">2. DHCP 中继（DHCP Relay）的工作原理</font>
**<font style="color:rgb(15, 17, 21);">为什么需要中继？</font>**<font style="color:rgb(15, 17, 21);">  
</font><font style="color:rgb(15, 17, 21);">DHCP 请求是广播帧，只在同一个广播域（VLAN）内传播。如果 DHCP 服务器不在同一个 VLAN，广播无法跨 VLAN 到达服务器。</font><font style="color:rgb(15, 17, 21);">  
</font>**<font style="color:rgb(15, 17, 21);">解决方法</font>**<font style="color:rgb(15, 17, 21);">：在网关设备（三层交换机或路由器）上启用</font><font style="color:rgb(15, 17, 21);"> </font>**<font style="color:rgb(15, 17, 21);">DHCP 中继</font>**<font style="color:rgb(15, 17, 21);">，将广播的 Discover 转换为</font>**<font style="color:rgb(15, 17, 21);">单播</font>**<font style="color:rgb(15, 17, 21);">发送给指定的 DHCP 服务器，同时填写</font><font style="color:rgb(15, 17, 21);"> </font>`**<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">giaddr</font>**`**<font style="color:rgb(15, 17, 21);">（Gateway IP Address）</font>**<font style="color:rgb(15, 17, 21);"> </font><font style="color:rgb(15, 17, 21);">字段，告诉服务器客户端所在的子网。</font>

**<font style="color:rgb(15, 17, 21);">中继过程</font>**<font style="color:rgb(15, 17, 21);">：</font>

1. <font style="color:rgb(15, 17, 21);">客户端发送</font><font style="color:rgb(15, 17, 21);"> </font>**<font style="color:rgb(15, 17, 21);">Discover（广播）</font>**<font style="color:rgb(15, 17, 21);">。</font>
2. <font style="color:rgb(15, 17, 21);">中继设备（SW3）收到后，将源 IP 改为自己的 VLANIF 接口 IP（如</font><font style="color:rgb(15, 17, 21);"> </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">192.168.10.1</font>`<font style="color:rgb(15, 17, 21);">），目的 IP 改为 DHCP 服务器 IP（</font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">13.0.0.1</font>`<font style="color:rgb(15, 17, 21);">），并填充</font><font style="color:rgb(15, 17, 21);"> </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">giaddr = 192.168.10.1</font>`<font style="color:rgb(15, 17, 21);">，然后单播转发。</font>
3. <font style="color:rgb(15, 17, 21);">DHCP 服务器根据</font><font style="color:rgb(15, 17, 21);"> </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">giaddr</font>`<font style="color:rgb(15, 17, 21);"> </font><font style="color:rgb(15, 17, 21);">确定客户端所属网段，从对应的地址池中分配 IP。</font>
4. <font style="color:rgb(15, 17, 21);">服务器回复</font><font style="color:rgb(15, 17, 21);"> </font>**<font style="color:rgb(15, 17, 21);">Offer</font>**<font style="color:rgb(15, 17, 21);"> </font><font style="color:rgb(15, 17, 21);">给中继的</font><font style="color:rgb(15, 17, 21);"> </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">giaddr</font>`<font style="color:rgb(15, 17, 21);">，中继再广播/单播给客户端。</font>

**<font style="color:rgb(15, 17, 21);">关键字段</font>**<font style="color:rgb(15, 17, 21);">：</font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">giaddr</font>`<font style="color:rgb(15, 17, 21);">（Gateway Interface Address）——中继设备的接口 IP，服务器用它来匹配地址池。</font>

### <font style="color:rgb(15, 17, 21);">3. 你的配置解析与补充</font>
#### <font style="color:rgb(15, 17, 21);">R1 上的 DHCP 配置</font>
<font style="color:rgb(15, 17, 21);">bash</font>

```plain
[R1] dhcp enable                     # 全局开启 DHCP 服务
[R1] interface GigabitEthernet0/0/0
[R1-GigabitEthernet0/0/0] ip address 172.16.10.1 24
[R1-GigabitEthernet0/0/0] dhcp select interface   # 接口地址池模式，自动使用接口网段分配 IP
[R1-GigabitEthernet0/0/0] quit
[R1] interface GigabitEthernet0/0/1
[R1-GigabitEthernet0/0/1] ip address 172.16.20.1 24
[R1-GigabitEthernet0/0/1] dhcp select interface
```

**<font style="color:rgb(15, 17, 21);">注意</font>**<font style="color:rgb(15, 17, 21);">：你写的是</font><font style="color:rgb(15, 17, 21);"> </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">dhcp selsect int</font>`<font style="color:rgb(15, 17, 21);">，正确的命令是</font><font style="color:rgb(15, 17, 21);"> </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">dhcp select interface</font>`<font style="color:rgb(15, 17, 21);">。</font>

#### <font style="color:rgb(15, 17, 21);">全局地址池（为 RoomC~G 准备）</font>
<font style="color:rgb(15, 17, 21);">bash</font>

```plain
[R1] ip pool roomC
[R1-ip-pool-roomC] gateway-list 192.168.10.1
[R1-ip-pool-roomC] network 192.168.10.0 mask 24
...（roomD~roomG 类似）
```

<font style="color:rgb(15, 17, 21);">这些地址池</font>**<font style="color:rgb(15, 17, 21);">并没有被任何接口直接使用</font>**<font style="color:rgb(15, 17, 21);">，而是等待 DHCP 中继发来的请求通过</font><font style="color:rgb(15, 17, 21);"> </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">giaddr</font>`<font style="color:rgb(15, 17, 21);"> </font><font style="color:rgb(15, 17, 21);">匹配。</font>

#### <font style="color:rgb(15, 17, 21);">R1 连接 SW3 的接口</font>
<font style="color:rgb(15, 17, 21);">bash</font>

```plain
[R1] interface GigabitEthernet0/0/2
[R1-GigabitEthernet0/0/2] ip address 13.0.0.1 24
[R1-GigabitEthernet0/0/2] dhcp select global   # 启用全局地址池，处理来自中继的请求
```

**<font style="color:rgb(15, 17, 21);">关键</font>**<font style="color:rgb(15, 17, 21);">：</font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">dhcp select global</font>`<font style="color:rgb(15, 17, 21);"> </font><font style="color:rgb(15, 17, 21);">表示该接口收到 DHCP 请求后，使用</font>**<font style="color:rgb(15, 17, 21);">全局地址池</font>**<font style="color:rgb(15, 17, 21);">（而不是接口地址池）。当中继转发的请求到达</font><font style="color:rgb(15, 17, 21);"> </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">13.0.0.1</font>`<font style="color:rgb(15, 17, 21);"> </font><font style="color:rgb(15, 17, 21);">时，R1 会根据报文中的</font><font style="color:rgb(15, 17, 21);"> </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">giaddr</font>`<font style="color:rgb(15, 17, 21);"> </font><font style="color:rgb(15, 17, 21);">来匹配对应的</font><font style="color:rgb(15, 17, 21);"> </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">ip pool</font>`<font style="color:rgb(15, 17, 21);">（例如</font><font style="color:rgb(15, 17, 21);"> </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">giaddr=192.168.10.1</font>`<font style="color:rgb(15, 17, 21);"> </font><font style="color:rgb(15, 17, 21);">匹配</font><font style="color:rgb(15, 17, 21);"> </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">roomC</font>`<font style="color:rgb(15, 17, 21);">）。</font>

#### <font style="color:rgb(15, 17, 21);">SW3 上的 DHCP 中继配置</font>
<font style="color:rgb(15, 17, 21);">bash</font>

```plain
[SW3] dhcp enable
[SW3] interface Vlanif10
[SW3-Vlanif10] ip address 192.168.10.1 24
[SW3-Vlanif10] dhcp select relay            # 开启中继模式
[SW3-Vlanif10] dhcp relay server-ip 13.0.0.1 # 指定 DHCP 服务器地址
```

<font style="color:rgb(15, 17, 21);">同样为 Vlanif20~50 做相同配置。</font><font style="color:rgb(15, 17, 21);">  
</font>**<font style="color:rgb(15, 17, 21);">注意</font>**<font style="color:rgb(15, 17, 21);">：你写的是</font><font style="color:rgb(15, 17, 21);"> </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">dhcp select relay</font>`<font style="color:rgb(15, 17, 21);">，正确；</font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">dhcp relay server-ip</font>`<font style="color:rgb(15, 17, 21);"> </font><font style="color:rgb(15, 17, 21);">也是正确的。</font>

**<font style="color:rgb(15, 17, 21);">抓包验证中继</font>**<font style="color:rgb(15, 17, 21);">：在 SW3 连接 R1 的链路上抓包，过滤</font><font style="color:rgb(15, 17, 21);"> </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">udp.port==67</font>`<font style="color:rgb(15, 17, 21);">。你会看到 Discover 报文的源 IP 是</font><font style="color:rgb(15, 17, 21);"> </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">192.168.10.1</font>`<font style="color:rgb(15, 17, 21);">（Vlanif10 的 IP），目的 IP 是</font><font style="color:rgb(15, 17, 21);"> </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">13.0.0.1</font>`<font style="color:rgb(15, 17, 21);">，并且</font><font style="color:rgb(15, 17, 21);"> </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">giaddr</font>`<font style="color:rgb(15, 17, 21);"> </font><font style="color:rgb(15, 17, 21);">字段为</font><font style="color:rgb(15, 17, 21);"> </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">192.168.10.1</font>`<font style="color:rgb(15, 17, 21);">。</font>

---

## <font style="color:rgb(15, 17, 21);">三、OSPF 邻居建立过程与配置</font>
### <font style="color:rgb(15, 17, 21);">1. OSPF 邻居状态机</font>
<font style="color:rgb(15, 17, 21);">OSPF 使用</font><font style="color:rgb(15, 17, 21);"> </font>**<font style="color:rgb(15, 17, 21);">Hello 报文</font>**<font style="color:rgb(15, 17, 21);"> </font><font style="color:rgb(15, 17, 21);">发现和维护邻居，经过以下状态：</font>

| <font style="color:rgb(15, 17, 21);">状态</font> | <font style="color:rgb(15, 17, 21);">含义</font> | <font style="color:rgb(15, 17, 21);">发生的报文</font> |
| --- | --- | --- |
| **<font style="color:rgb(15, 17, 21);">Down</font>** | <font style="color:rgb(15, 17, 21);">未收到邻居的 Hello</font> | <font style="color:rgb(15, 17, 21);">—</font> |
| **<font style="color:rgb(15, 17, 21);">Init</font>** | <font style="color:rgb(15, 17, 21);">收到邻居的 Hello，但报文中没有自己的 Router ID</font> | <font style="color:rgb(15, 17, 21);">Hello</font> |
| **<font style="color:rgb(15, 17, 21);">2-Way</font>** | <font style="color:rgb(15, 17, 21);">收到 Hello 且报文中包含自己的 Router ID，邻居关系建立（广播网在此选举 DR/BDR）</font> | <font style="color:rgb(15, 17, 21);">Hello</font> |
| **<font style="color:rgb(15, 17, 21);">ExStart</font>** | <font style="color:rgb(15, 17, 21);">开始交换 DD 报文，协商主从（Router ID 大的为主）</font> | <font style="color:rgb(15, 17, 21);">DD（空）</font> |
| **<font style="color:rgb(15, 17, 21);">Exchange</font>** | <font style="color:rgb(15, 17, 21);">交换 DD 报文描述 LSDB 摘要</font> | <font style="color:rgb(15, 17, 21);">DD（含 LSA 头）</font> |
| **<font style="color:rgb(15, 17, 21);">Loading</font>** | <font style="color:rgb(15, 17, 21);">通过 LSR 请求缺失的 LSA，通过 LSU 发送完整 LSA</font> | <font style="color:rgb(15, 17, 21);">LSR, LSU, LSAck</font> |
| **<font style="color:rgb(15, 17, 21);">Full</font>** | <font style="color:rgb(15, 17, 21);">LSDB 完全同步，邻接关系建立</font> | <font style="color:rgb(15, 17, 21);">—</font> |


### <font style="color:rgb(15, 17, 21);">2. 关键报文详解</font>
+ **<font style="color:rgb(15, 17, 21);">Hello</font>**<font style="color:rgb(15, 17, 21);">：周期发送（默认 10 秒），包含 Router ID、Area ID、优先级、Hello/Dead 间隔、邻居列表等。</font>
+ **<font style="color:rgb(15, 17, 21);">DD（Database Description）</font>**<font style="color:rgb(15, 17, 21);">：描述 LSDB 中 LSA 的摘要（每个 LSA 的头信息）。</font>
+ **<font style="color:rgb(15, 17, 21);">LSR（Link State Request）</font>**<font style="color:rgb(15, 17, 21);">：请求邻居的某个具体 LSA。</font>
+ **<font style="color:rgb(15, 17, 21);">LSU（Link State Update）</font>**<font style="color:rgb(15, 17, 21);">：发送完整的 LSA 内容。</font>
+ **<font style="color:rgb(15, 17, 21);">LSAck（Link State Acknowledgment）</font>**<font style="color:rgb(15, 17, 21);">：确认收到 LSU。</font>

**<font style="color:rgb(15, 17, 21);">抓包示例</font>**<font style="color:rgb(15, 17, 21);">：在 R1 和 SW3 之间的链路抓包，过滤</font><font style="color:rgb(15, 17, 21);"> </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">ospf</font>`<font style="color:rgb(15, 17, 21);">。你会看到 Hello 报文（每 10 秒一次），然后当邻居首次建立时，会有 DD、LSR、LSU 的交换过程。</font>

### <font style="color:rgb(15, 17, 21);">3. 你的 OSPF 配置解析</font>
#### <font style="color:rgb(15, 17, 21);">R1 的 OSPF</font>
<font style="color:rgb(15, 17, 21);">bash</font>

```plain
[R1] ospf 1 router-id 1.1.1.1
[R1-ospf-1] area 0
[R1-ospf-1-area-0.0.0.0] network 13.0.0.1 0.0.0.255
```

<font style="color:rgb(15, 17, 21);">你写的是</font><font style="color:rgb(15, 17, 21);"> </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">ospf 1 rout 1.1.1.1</font>`<font style="color:rgb(15, 17, 21);">，正确的是</font><font style="color:rgb(15, 17, 21);"> </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">ospf 1 router-id 1.1.1.1</font>`<font style="color:rgb(15, 17, 21);">。</font><font style="color:rgb(15, 17, 21);">  
</font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">network 13.0.0.1 0.0.0.255</font>`<font style="color:rgb(15, 17, 21);"> </font><font style="color:rgb(15, 17, 21);">宣告的是</font><font style="color:rgb(15, 17, 21);"> </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">13.0.0.0/24</font>`<font style="color:rgb(15, 17, 21);"> </font><font style="color:rgb(15, 17, 21);">整个网段（反掩码</font><font style="color:rgb(15, 17, 21);"> </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">0.0.0.255</font>`<font style="color:rgb(15, 17, 21);"> </font><font style="color:rgb(15, 17, 21);">表示前 24 位固定）。你也可以用精确宣告</font><font style="color:rgb(15, 17, 21);"> </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">network 13.0.0.1 0.0.0.0</font>`<font style="color:rgb(15, 17, 21);">，但通常建议宣告整个直连网段。</font>

#### <font style="color:rgb(15, 17, 21);">SW3 的 OSPF</font>
<font style="color:rgb(15, 17, 21);">bash</font>

```plain
[SW3] ospf 1 router-id 3.3.3.3
[SW3-ospf-1] area 0
[SW3-ospf-1-area-0.0.0.0] network 13.0.0.3 0.0.0.255
```

**<font style="color:rgb(15, 17, 21);">注意</font>**<font style="color:rgb(15, 17, 21);">：SW3 只需要宣告</font><font style="color:rgb(15, 17, 21);"> </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">13.0.0.0/24</font>`<font style="color:rgb(15, 17, 21);"> </font><font style="color:rgb(15, 17, 21);">网段，它不需要宣告 VLANIF 10~50 的网段，因为这些网段是直连的，OSPF 会自动学习到直连路由吗？</font>**<font style="color:rgb(15, 17, 21);">不会</font>**<font style="color:rgb(15, 17, 21);">！OSPF 只宣告通过</font><font style="color:rgb(15, 17, 21);"> </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">network</font>`<font style="color:rgb(15, 17, 21);"> </font><font style="color:rgb(15, 17, 21);">命令指定的接口。为了让 OSPF 将这些 VLAN 网段通告给 R1，你需要在 SW3 上也宣告它们，例如：</font>

<font style="color:rgb(15, 17, 21);">bash</font>

```plain
[SW3-ospf-1-area-0.0.0.0] network 192.168.10.0 0.0.0.255
[SW3-ospf-1-area-0.0.0.0] network 192.168.20.0 0.0.0.255
... 等等
```

<font style="color:rgb(15, 17, 21);">否则 R1 无法学习到这些网段的路由，PC 也无法访问 R1 的直连网段（如</font><font style="color:rgb(15, 17, 21);"> </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">172.16.10.0/24</font>`<font style="color:rgb(15, 17, 21);">）。</font>**<font style="color:rgb(15, 17, 21);">你的配置中缺少这一部分</font>**<font style="color:rgb(15, 17, 21);">，需要补充。</font>

### <font style="color:rgb(15, 17, 21);">4. DR/BDR 选举（广播网络）</font>
<font style="color:rgb(15, 17, 21);">在以太网链路上（如 R1 和 SW3 之间的</font><font style="color:rgb(15, 17, 21);"> </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">13.0.0.0/24</font>`<font style="color:rgb(15, 17, 21);">），OSPF 默认网络类型为</font><font style="color:rgb(15, 17, 21);"> </font>**<font style="color:rgb(15, 17, 21);">Broadcast</font>**<font style="color:rgb(15, 17, 21);">，会选举 DR（Designated Router）和 BDR（Backup DR）。选举规则：</font>

+ <font style="color:rgb(15, 17, 21);">优先级（</font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">ospf dr-priority</font>`<font style="color:rgb(15, 17, 21);">，默认 1）越高越优。</font>
+ <font style="color:rgb(15, 17, 21);">优先级相同，Router ID 越大越优。</font>
+ <font style="color:rgb(15, 17, 21);">优先级为 0 不参与选举。</font>

<font style="color:rgb(15, 17, 21);">你的配置中没有设置优先级，所以默认都是 1，那么 Router ID 大的会成为 DR。R1 的 Router ID 是</font><font style="color:rgb(15, 17, 21);"> </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">1.1.1.1</font>`<font style="color:rgb(15, 17, 21);">，SW3 的是</font><font style="color:rgb(15, 17, 21);"> </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">3.3.3.3</font>`<font style="color:rgb(15, 17, 21);">，因此</font><font style="color:rgb(15, 17, 21);"> </font>**<font style="color:rgb(15, 17, 21);">SW3 会成为 DR，R1 成为 BDR</font>**<font style="color:rgb(15, 17, 21);">（如果只有两台路由器，BDR 也会存在）。你可以通过</font><font style="color:rgb(15, 17, 21);"> </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">display ospf interface</font>`<font style="color:rgb(15, 17, 21);"> </font><font style="color:rgb(15, 17, 21);">查看。</font>

---

## <font style="color:rgb(15, 17, 21);">四、VLAN 与三层交换基础</font>
### <font style="color:rgb(15, 17, 21);">1. VLAN 划分与端口类型</font>
+ **<font style="color:rgb(15, 17, 21);">Access 端口</font>**<font style="color:rgb(15, 17, 21);">：连接终端（PC、服务器），只属于一个 VLAN。收无标记帧时打上 PVID 标签；发送时剥离标签。</font>
+ **<font style="color:rgb(15, 17, 21);">Trunk 端口</font>**<font style="color:rgb(15, 17, 21);">：连接交换机，允许多个 VLAN 通过，帧带 802.1Q 标签。</font>
+ **<font style="color:rgb(15, 17, 21);">Hybrid 端口</font>**<font style="color:rgb(15, 17, 21);">：类似 Trunk，但可控制哪些 VLAN 带标签、哪些不带（较少用）。</font>

<font style="color:rgb(15, 17, 21);">你的 SW3 上，连接 PC 的端口都配置为</font><font style="color:rgb(15, 17, 21);"> </font>**<font style="color:rgb(15, 17, 21);">Access</font>**<font style="color:rgb(15, 17, 21);"> </font><font style="color:rgb(15, 17, 21);">并指定了对应的 VLAN，正确。连接 R1 的端口（</font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">g0/0/1</font>`<font style="color:rgb(15, 17, 21);">）你配置为 Access VLAN 13，这没问题，因为 R1 和 SW3 之间是三层链路，不需要 Trunk。</font>

### <font style="color:rgb(15, 17, 21);">2. VLANIF（SVI）接口</font>
<font style="color:rgb(15, 17, 21);">VLANIF 是三层交换机的逻辑接口，用于实现 VLAN 间路由或连接外部网络。配置 IP 后，该 IP 就成为该 VLAN 内主机的</font>**<font style="color:rgb(15, 17, 21);">默认网关</font>**<font style="color:rgb(15, 17, 21);">。</font>

<font style="color:rgb(15, 17, 21);">你已经在 SW3 上为 VLAN 10,20,30,40,50,13 配置了 VLANIF 接口，并分配了 IP。其中 VLANIF 13 的 IP</font><font style="color:rgb(15, 17, 21);"> </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">13.0.0.3/24</font>`<font style="color:rgb(15, 17, 21);"> </font><font style="color:rgb(15, 17, 21);">与 R1 的</font><font style="color:rgb(15, 17, 21);"> </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">13.0.0.1/24</font>`<font style="color:rgb(15, 17, 21);"> </font><font style="color:rgb(15, 17, 21);">在同一网段，它们通过二层交换机（或直连）相连，形成三层互通。</font>

---

## <font style="color:rgb(15, 17, 21);">五、补充与修正命令清单（不是完整脚本）</font>
**<font style="color:rgb(15, 17, 21);">修正 R1 的 DHCP 命令</font>**<font style="color:rgb(15, 17, 21);">：</font>

1. <font style="color:rgb(15, 17, 21);">bash </font>

<font style="color:rgb(15, 17, 21);"> dhcp select interface   # 而不是 selsect int</font>

**<font style="color:rgb(15, 17, 21);">在 R1 上，为 RoomC~G 的地址池配置 DNS 和租期（可选）</font>**<font style="color:rgb(15, 17, 21);">：</font>

2. <font style="color:rgb(15, 17, 21);">bash</font>

```plain
ip pool roomC
dns-list 114.114.114.114
lease day 3
```

**<font style="color:rgb(15, 17, 21);">在 SW3 上，将 VLAN 10~50 的网段宣告进 OSPF</font>**<font style="color:rgb(15, 17, 21);">：</font>

3. <font style="color:rgb(15, 17, 21);">bash</font>

```plain
ospf 1
area 0
 network 192.168.10.0 0.0.0.255
 network 192.168.20.0 0.0.0.255
 network 192.168.30.0 0.0.0.255
 network 192.168.40.0 0.0.0.255
 network 192.168.50.0 0.0.0.255
```

4. **<font style="color:rgb(15, 17, 21);">确保 SW3 连接 PC 的端口都在正确的 VLAN 中</font>**<font style="color:rgb(15, 17, 21);">（你已配置，但可加</font><font style="color:rgb(15, 17, 21);"> </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">stp edged-port enable</font>`<font style="color:rgb(15, 17, 21);"> </font><font style="color:rgb(15, 17, 21);">加速收敛）。</font>
5. **<font style="color:rgb(15, 17, 21);">检查 R1 和 SW3 的物理接口状态</font>**<font style="color:rgb(15, 17, 21);">，确保</font><font style="color:rgb(15, 17, 21);"> </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">13.0.0.1</font>`<font style="color:rgb(15, 17, 21);"> </font><font style="color:rgb(15, 17, 21);">和</font><font style="color:rgb(15, 17, 21);"> </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">13.0.0.3</font>`<font style="color:rgb(15, 17, 21);"> </font><font style="color:rgb(15, 17, 21);">能 ping 通。</font>

---

## <font style="color:rgb(15, 17, 21);">六、验证与抓包建议</font>
+ **<font style="color:rgb(15, 17, 21);">DHCP 中继验证</font>**<font style="color:rgb(15, 17, 21);">：在 SW3 的 VLANIF 10 接口上抓包，过滤</font><font style="color:rgb(15, 17, 21);"> </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">bootp</font>`<font style="color:rgb(15, 17, 21);">，观察 Discover 报文是否被单播转发到</font><font style="color:rgb(15, 17, 21);"> </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">13.0.0.1</font>`<font style="color:rgb(15, 17, 21);">，以及 Offer 是否返回。</font>
+ **<font style="color:rgb(15, 17, 21);">OSPF 邻居验证</font>**<font style="color:rgb(15, 17, 21);">：</font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">display ospf peer</font>`<font style="color:rgb(15, 17, 21);"> </font><font style="color:rgb(15, 17, 21);">查看邻居状态是否为 Full。若卡在 ExStart，检查 MTU 是否一致；卡在 2-Way 可能是 DR 选举未完成。</font>
+ **<font style="color:rgb(15, 17, 21);">路由验证</font>**<font style="color:rgb(15, 17, 21);">：</font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">display ip routing-table</font>`<font style="color:rgb(15, 17, 21);">，R1 应能看到</font><font style="color:rgb(15, 17, 21);"> </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">192.168.10.0/24</font>`<font style="color:rgb(15, 17, 21);"> </font><font style="color:rgb(15, 17, 21);">等网段，SW3 应能看到</font><font style="color:rgb(15, 17, 21);"> </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">172.16.10.0/24</font>`<font style="color:rgb(15, 17, 21);"> </font><font style="color:rgb(15, 17, 21);">等网段。</font>
+ **<font style="color:rgb(15, 17, 21);">PC 获取 IP</font>**<font style="color:rgb(15, 17, 21);">：在 PC 上执行</font><font style="color:rgb(15, 17, 21);"> </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">ipconfig</font>`<font style="color:rgb(15, 17, 21);">，应获得对应网段的 IP，且能 ping 通网关和 R1 的</font><font style="color:rgb(15, 17, 21);"> </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">172.16.10.1</font>`<font style="color:rgb(15, 17, 21);">。</font>

---

## <font style="color:rgb(15, 17, 21);">七、英文缩写总结</font>
| <font style="color:rgb(15, 17, 21);">缩写</font> | <font style="color:rgb(15, 17, 21);">全称</font> | <font style="color:rgb(15, 17, 21);">含义</font> |
| --- | --- | --- |
| <font style="color:rgb(15, 17, 21);">DHCP</font> | <font style="color:rgb(15, 17, 21);">Dynamic Host Configuration Protocol</font> | <font style="color:rgb(15, 17, 21);">动态主机配置协议</font> |
| <font style="color:rgb(15, 17, 21);">DORA</font> | <font style="color:rgb(15, 17, 21);">Discover, Offer, Request, Ack</font> | <font style="color:rgb(15, 17, 21);">DHCP 四个交互报文</font> |
| <font style="color:rgb(15, 17, 21);">giaddr</font> | <font style="color:rgb(15, 17, 21);">Gateway Interface Address</font> | <font style="color:rgb(15, 17, 21);">DHCP 中继填充的网关地址</font> |
| <font style="color:rgb(15, 17, 21);">OSPF</font> | <font style="color:rgb(15, 17, 21);">Open Shortest Path First</font> | <font style="color:rgb(15, 17, 21);">开放最短路径优先</font> |
| <font style="color:rgb(15, 17, 21);">LSA</font> | <font style="color:rgb(15, 17, 21);">Link State Advertisement</font> | <font style="color:rgb(15, 17, 21);">链路状态通告</font> |
| <font style="color:rgb(15, 17, 21);">LSDB</font> | <font style="color:rgb(15, 17, 21);">Link State Database</font> | <font style="color:rgb(15, 17, 21);">链路状态数据库</font> |
| <font style="color:rgb(15, 17, 21);">LSU</font> | <font style="color:rgb(15, 17, 21);">Link State Update</font> | <font style="color:rgb(15, 17, 21);">链路状态更新</font> |
| <font style="color:rgb(15, 17, 21);">LSR</font> | <font style="color:rgb(15, 17, 21);">Link State Request</font> | <font style="color:rgb(15, 17, 21);">链路状态请求</font> |
| <font style="color:rgb(15, 17, 21);">DR</font> | <font style="color:rgb(15, 17, 21);">Designated Router</font> | <font style="color:rgb(15, 17, 21);">指定路由器</font> |
| <font style="color:rgb(15, 17, 21);">BDR</font> | <font style="color:rgb(15, 17, 21);">Backup Designated Router</font> | <font style="color:rgb(15, 17, 21);">备份指定路由器</font> |
| <font style="color:rgb(15, 17, 21);">VLAN</font> | <font style="color:rgb(15, 17, 21);">Virtual Local Area Network</font> | <font style="color:rgb(15, 17, 21);">虚拟局域网</font> |
| <font style="color:rgb(15, 17, 21);">VLANIF</font> | <font style="color:rgb(15, 17, 21);">VLAN Interface</font> | <font style="color:rgb(15, 17, 21);">三层交换机的 VLAN 虚接口</font> |
| <font style="color:rgb(15, 17, 21);">PVID</font> | <font style="color:rgb(15, 17, 21);">Port VLAN ID</font> | <font style="color:rgb(15, 17, 21);">端口默认 VLAN ID</font> |



