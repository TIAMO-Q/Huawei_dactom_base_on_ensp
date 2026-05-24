<img src="https://cdn.nlark.com/yuque/0/2026/png/50508853/1776508652838-e2ddaa56-8d9e-4739-8baf-40a21c3230a4.png" width="1330.6666666666667" title="" crop="0,0,1,1" id="ue5f99421" class="ne-image">

## <font style="color:rgb(15, 17, 21);">一、拓扑分析与当前配置解读</font>
### <font style="color:rgb(15, 17, 21);">1. 拓扑结构（根据图片和命令反推）</font>
+ **<font style="color:rgb(15, 17, 21);">接入层</font>**<font style="color:rgb(15, 17, 21);">：LSW1（二层交换机），划分 VLAN 10、20，分别连接 PC1、PC2。</font>
+ **<font style="color:rgb(15, 17, 21);">汇聚层</font>**<font style="color:rgb(15, 17, 21);">：LSW2（三层交换机），VLAN 10、20 的网关（VLANIF 10、20），同时 VLAN 23 用于连接 LSW3。</font>
+ **<font style="color:rgb(15, 17, 21);">核心层</font>**<font style="color:rgb(15, 17, 21);">：LSW3（三层交换机），VLAN 23 连接 LSW2，VLAN 13 连接 R1。</font>
+ **<font style="color:rgb(15, 17, 21);">出口路由器</font>**<font style="color:rgb(15, 17, 21);">：R1，单臂路由子接口连接 LSW4（VLAN 100、200），并连接服务器 Server1（192.168.100.0/24）、Server2（192.168.200.0/24）。</font>
+ **<font style="color:rgb(15, 17, 21);">OSPF</font>**<font style="color:rgb(15, 17, 21);">：所有三层设备（SW2、SW3、R1）运行 OSPF Area 0，实现全网路由互通。</font>

### <font style="color:rgb(15, 17, 21);">2. 当前配置分析</font>
<font style="color:rgb(15, 17, 21);">你已配置：</font>

+ <font style="color:rgb(15, 17, 21);">SW1：VLAN 10、20，Access/Trunk 正确。</font>
+ <font style="color:rgb(15, 17, 21);">SW2：VLAN 10、20、23，VLANIF 10/20/23，DHCP，OSPF 宣告三个网段。</font>
+ <font style="color:rgb(15, 17, 21);">SW3：VLAN 23、13，VLANIF 23/13，OSPF 宣告两个网段。</font>
+ <font style="color:rgb(15, 17, 21);">R1：物理接口 GE0/0/0 配 IP 192.168.13.1/24，子接口 GE0/0/1.100 和 .200 封装 VLAN 100/200，并配置 IP 作为网关，OSPF 宣告 192.168.13.1、192.168.100.254、192.168.200.254 以及 Loopback 1.1.1.1。</font>
+ <font style="color:rgb(15, 17, 21);">SW4：VLAN 100、200，Access 端口分别连接 Server1 和 Server2，Trunk 连接 R1。</font>

**<font style="color:rgb(15, 17, 21);">存在的问题/不足</font>**<font style="color:rgb(15, 17, 21);">：</font>

+ <font style="color:rgb(15, 17, 21);">SW2 的 VLANIF 10/20 已配置 DHCP，但 PC 需要能够获取 IP：需要确保 SW1 与 SW2 之间的 Trunk 允许 VLAN 10/20，且 PC 的 DHCP 请求能到达 SW2（广播域内，无需中继）。</font>
+ <font style="color:rgb(15, 17, 21);">SW3 的 VLANIF 23 与 SW2 的 VLANIF 23 三层互通（已通过 OSPF 学习），但物理连线是否正确？需要确保 SW2 的 G0/0/2（Access VLAN 23）与 SW3 的 G0/0/1（Access VLAN 23）二层直连。</font>
+ <font style="color:rgb(15, 17, 21);">R1 的子接口配置缺少</font><font style="color:rgb(15, 17, 21);"> </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">arp broadcast enable</font>`<font style="color:rgb(15, 17, 21);">（你已经写了</font><font style="color:rgb(15, 17, 21);"> </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">arp b en</font>`<font style="color:rgb(15, 17, 21);">，正确）。</font>
+ <font style="color:rgb(15, 17, 21);">SW4 的 Trunk 口需要允许 VLAN 100、200，且 R1 的子接口 VLAN ID 必须与 SW4 的 VLAN 对应。</font>

---

## <font style="color:rgb(15, 17, 21);">二、核心知识点详解</font>
### <font style="color:rgb(15, 17, 21);">1. 二层交换与 VLAN（Access / Trunk）</font>
+ **<font style="color:rgb(15, 17, 21);">Access 接口</font>**<font style="color:rgb(15, 17, 21);">：连接终端（PC、服务器），只属于一个 VLAN。收到无标记帧时打上 PVID 标签，发送时剥离标签。</font>
+ **<font style="color:rgb(15, 17, 21);">Trunk 接口</font>**<font style="color:rgb(15, 17, 21);">：连接交换机之间，允许多个 VLAN 通过，帧带 802.1Q 标签。</font>
+ **<font style="color:rgb(15, 17, 21);">PVID</font>**<font style="color:rgb(15, 17, 21);">：端口的默认 VLAN ID。Access 端口 PVID 就是所属 VLAN；Trunk 端口 PVID 默认 1，但可修改。</font>

**<font style="color:rgb(15, 17, 21);">配置命令</font>**<font style="color:rgb(15, 17, 21);">：</font>

+ <font style="color:rgb(15, 17, 21);">bash</font>

```plain
port link-type access
port default vlan 10
port link-type trunk
port trunk allow-pass vlan 10 20
```

### <font style="color:rgb(15, 17, 21);">2. 三层交换与 VLANIF（SVI）</font>
+ **<font style="color:rgb(15, 17, 21);">VLANIF</font>**<font style="color:rgb(15, 17, 21);">：三层交换机的逻辑接口，相当于每个 VLAN 的网关。</font>
+ **<font style="color:rgb(15, 17, 21);">作用</font>**<font style="color:rgb(15, 17, 21);">：实现 VLAN 间路由，以及连接外部网络。</font>

**<font style="color:rgb(15, 17, 21);">配置</font>**<font style="color:rgb(15, 17, 21);">：</font>

+ <font style="color:rgb(15, 17, 21);">bash</font>

```plain
interface Vlanif10
 ip address 192.168.10.1 24
```

+ **<font style="color:rgb(15, 17, 21);">前提</font>**<font style="color:rgb(15, 17, 21);">：该 VLAN 必须存在，且至少有一个 Access 端口处于 UP 状态，否则 VLANIF 会 down。</font>

### <font style="color:rgb(15, 17, 21);">3. 单臂路由（Router on a Stick）</font>
+ **<font style="color:rgb(15, 17, 21);">场景</font>**<font style="color:rgb(15, 17, 21);">：路由器只有一个物理接口连接交换机，需要为多个 VLAN 提供网关。</font>
+ **<font style="color:rgb(15, 17, 21);">原理</font>**<font style="color:rgb(15, 17, 21);">：在物理接口上创建多个子接口，每个子接口封装一个 VLAN ID，并配置 IP 作为该 VLAN 的网关。</font>
+ **<font style="color:rgb(15, 17, 21);">必须开启</font>**<font style="color:rgb(15, 17, 21);">：</font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">dot1q termination vid <vlan-id></font>`<font style="color:rgb(15, 17, 21);"> </font><font style="color:rgb(15, 17, 21);">和</font><font style="color:rgb(15, 17, 21);"> </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">arp broadcast enable</font>`<font style="color:rgb(15, 17, 21);">（华为）。</font>
+ **<font style="color:rgb(15, 17, 21);">交换机侧</font>**<font style="color:rgb(15, 17, 21);">：连接路由器的接口必须为 Trunk，允许相关 VLAN 通过。</font>

**<font style="color:rgb(15, 17, 21);">配置示例</font>**<font style="color:rgb(15, 17, 21);">：</font>

+ <font style="color:rgb(15, 17, 21);">bash</font>

```plain
interface GigabitEthernet0/0/1.100
 dot1q termination vid 100
 ip address 192.168.100.254 24
 arp broadcast enable
```

### <font style="color:rgb(15, 17, 21);">4. OSPF（开放最短路径优先）</font>
+ **<font style="color:rgb(15, 17, 21);">Router ID</font>**<font style="color:rgb(15, 17, 21);">：唯一标识，建议手动指定。</font>
+ **<font style="color:rgb(15, 17, 21);">区域（Area）</font>**<font style="color:rgb(15, 17, 21);">：Area 0 为骨干区域，所有非骨干必须直连 Area 0。</font>
+ **<font style="color:rgb(15, 17, 21);">Network 宣告</font>**<font style="color:rgb(15, 17, 21);">：使用反掩码（wildcard mask），例如</font><font style="color:rgb(15, 17, 21);"> </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">192.168.10.0 0.0.0.255</font>`<font style="color:rgb(15, 17, 21);"> </font><font style="color:rgb(15, 17, 21);">表示 /24 网段。也可以精确宣告接口 IP（</font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">192.168.10.1 0.0.0.0</font>`<font style="color:rgb(15, 17, 21);">），但推荐宣告整个网段。</font>
+ **<font style="color:rgb(15, 17, 21);">LSA 类型</font>**<font style="color:rgb(15, 17, 21);">：Type 1（路由器）、Type 2（网络）、Type 3（区域间汇总）、Type 5（外部路由）等。</font>
+ **<font style="color:rgb(15, 17, 21);">查看命令</font>**<font style="color:rgb(15, 17, 21);">：</font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">display ospf peer</font>`<font style="color:rgb(15, 17, 21);">、</font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">display ospf lsdb</font>`<font style="color:rgb(15, 17, 21);">、</font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">display ip routing-table</font>`<font style="color:rgb(15, 17, 21);">。</font>

### <font style="color:rgb(15, 17, 21);">5. DHCP（动态主机配置协议）</font>
+ **<font style="color:rgb(15, 17, 21);">接口地址池模式</font>**<font style="color:rgb(15, 17, 21);">：</font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">dhcp select interface</font>`<font style="color:rgb(15, 17, 21);">，自动使用接口所在网段分配地址。</font>
+ **<font style="color:rgb(15, 17, 21);">全局地址池模式</font>**<font style="color:rgb(15, 17, 21);">：先创建</font><font style="color:rgb(15, 17, 21);"> </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">ip pool</font>`<font style="color:rgb(15, 17, 21);">，再在接口下</font><font style="color:rgb(15, 17, 21);"> </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">dhcp select global</font>`<font style="color:rgb(15, 17, 21);">。</font>
+ **<font style="color:rgb(15, 17, 21);">DHCP 中继</font>**<font style="color:rgb(15, 17, 21);">：当客户端和服务器不在同一广播域时，需要在网关设备上配置</font><font style="color:rgb(15, 17, 21);"> </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">dhcp select relay</font>`<font style="color:rgb(15, 17, 21);"> </font><font style="color:rgb(15, 17, 21);">+</font><font style="color:rgb(15, 17, 21);"> </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">dhcp relay server-ip</font>`<font style="color:rgb(15, 17, 21);">。</font>
+ **<font style="color:rgb(15, 17, 21);">你的实验</font>**<font style="color:rgb(15, 17, 21);">：SW2 上 VLANIF 10/20 使用接口地址池，PC 通过广播即可获取 IP，无需中继。</font>

---

## <font style="color:rgb(15, 17, 21);">三、需要补充/修正的配置</font>
### <font style="color:rgb(15, 17, 21);">1. SW2 的 DHCP 地址池配置（确保分配正确）</font>
<font style="color:rgb(15, 17, 21);">你已配置</font><font style="color:rgb(15, 17, 21);"> </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">dhcp select int</font>`<font style="color:rgb(15, 17, 21);">，但最好加上 DNS 和排除地址（可选）：</font>

<font style="color:rgb(15, 17, 21);">bash</font>

```plain
[SW2] interface Vlanif10
[SW2-Vlanif10] dhcp server dns-list 114.114.114.114
[SW2-Vlanif10] dhcp server excluded-ip-address 192.168.10.1 192.168.10.10
[SW2-Vlanif10] quit
[SW2] interface Vlanif20
[SW2-Vlanif20] dhcp server dns-list 114.114.114.114
[SW2-Vlanif20] dhcp server excluded-ip-address 192.168.20.1 192.168.20.10
```

### <font style="color:rgb(15, 17, 21);">2. SW3 的 VLAN 23 接口必须为 Access（你已配置正确）</font>
<font style="color:rgb(15, 17, 21);">但确保 SW2 的 G0/0/2 和 SW3 的 G0/0/1 都是 Access VLAN 23，且物理连线正确。</font>

### <font style="color:rgb(15, 17, 21);">3. R1 的 OSPF 宣告网段优化</font>
<font style="color:rgb(15, 17, 21);">你用了精确接口宣告（</font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">192.168.13.1 0.0.0.0</font>`<font style="color:rgb(15, 17, 21);">），也可以改为网段宣告：</font>

<font style="color:rgb(15, 17, 21);">bash</font>

```plain
[SW3] ospf 1
[SW3-ospf-1] area 0
[SW3-ospf-1-area-0.0.0.0] network 192.168.13.0 0.0.0.255
```

<font style="color:rgb(15, 17, 21);">但两种都可以。</font>

### <font style="color:rgb(15, 17, 21);">4. SW4 的 Trunk 配置确认</font>
<font style="color:rgb(15, 17, 21);">bash</font>

```plain
[SW4] interface GigabitEthernet0/0/1
[SW4-GigabitEthernet0/0/1] port link-type trunk
[SW4-GigabitEthernet0/0/1] port trunk allow-pass vlan 100 200
```

### <font style="color:rgb(15, 17, 21);">5. 验证 OSPF 邻居</font>
<font style="color:rgb(15, 17, 21);">在 SW2、SW3、R1 上分别执行：</font>

<font style="color:rgb(15, 17, 21);">bash</font>

<font style="color:rgb(15, 17, 21);">display ospf peer brief</font>

<font style="color:rgb(15, 17, 21);">应看到邻居状态为 Full。</font>

### <font style="color:rgb(15, 17, 21);">6. 验证 DHCP</font>
<font style="color:rgb(15, 17, 21);">在 PC1 和 PC2 上执行</font><font style="color:rgb(15, 17, 21);"> </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">ipconfig</font>`<font style="color:rgb(15, 17, 21);">，应能获得 192.168.10.x 和 192.168.20.x 的地址。</font>

### <font style="color:rgb(15, 17, 21);">7. 验证全网通</font>
+ <font style="color:rgb(15, 17, 21);">PC1 ping 192.168.100.254（R1 子接口网关）应通。</font>
+ <font style="color:rgb(15, 17, 21);">PC1 ping 192.168.200.254 应通。</font>
+ <font style="color:rgb(15, 17, 21);">PC1 ping 1.1.1.1（R1 Loopback）应通。</font>
+ <font style="color:rgb(15, 17, 21);">Server1（192.168.100.x） ping PC1 应通。</font>

---

## <font style="color:rgb(15, 17, 21);">四、你欠缺的火候——二三层与单臂路由知识总结</font>
### <font style="color:rgb(15, 17, 21);">二层转发 vs 三层路由</font>
| <font style="color:rgb(15, 17, 21);">项目</font> | <font style="color:rgb(15, 17, 21);">二层交换</font> | <font style="color:rgb(15, 17, 21);">三层路由</font> |
| --- | --- | --- |
| <font style="color:rgb(15, 17, 21);">依据</font> | <font style="color:rgb(15, 17, 21);">MAC 地址表</font> | <font style="color:rgb(15, 17, 21);">IP 路由表</font> |
| <font style="color:rgb(15, 17, 21);">范围</font> | <font style="color:rgb(15, 17, 21);">同一 VLAN / 广播域</font> | <font style="color:rgb(15, 17, 21);">不同网段 / 不同 VLAN</font> |
| <font style="color:rgb(15, 17, 21);">设备</font> | <font style="color:rgb(15, 17, 21);">二层交换机</font> | <font style="color:rgb(15, 17, 21);">路由器、三层交换机</font> |
| <font style="color:rgb(15, 17, 21);">VLAN 间通信</font> | <font style="color:rgb(15, 17, 21);">不可能</font> | <font style="color:rgb(15, 17, 21);">必须通过三层设备</font> |


### <font style="color:rgb(15, 17, 21);">单臂路由与三层交换的对比</font>
| <font style="color:rgb(15, 17, 21);">技术</font> | <font style="color:rgb(15, 17, 21);">优点</font> | <font style="color:rgb(15, 17, 21);">缺点</font> | <font style="color:rgb(15, 17, 21);">适用场景</font> |
| --- | --- | --- | --- |
| <font style="color:rgb(15, 17, 21);">单臂路由</font> | <font style="color:rgb(15, 17, 21);">无需三层交换机，成本低</font> | <font style="color:rgb(15, 17, 21);">路由器单点故障，带宽瓶颈</font> | <font style="color:rgb(15, 17, 21);">小规模网络，临时方案</font> |
| <font style="color:rgb(15, 17, 21);">VLANIF（SVI）</font> | <font style="color:rgb(15, 17, 21);">线速转发，高可靠</font> | <font style="color:rgb(15, 17, 21);">需要三层交换机</font> | <font style="color:rgb(15, 17, 21);">企业园区网核心</font> |


### <font style="color:rgb(15, 17, 21);">关键配置命令对比</font>
| <font style="color:rgb(15, 17, 21);">功能</font> | <font style="color:rgb(15, 17, 21);">单臂路由（路由器）</font> | <font style="color:rgb(15, 17, 21);">三层交换（VLANIF）</font> |
| --- | --- | --- |
| <font style="color:rgb(15, 17, 21);">创建 VLAN</font> | <font style="color:rgb(15, 17, 21);">无</font> | `<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">vlan batch 10 20</font>` |
| <font style="color:rgb(15, 17, 21);">网关配置</font> | <font style="color:rgb(15, 17, 21);">子接口</font><font style="color:rgb(15, 17, 21);"> </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">ip address 192.168.10.1 24</font>` | `<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">interface Vlanif10</font>`<br/><font style="color:rgb(15, 17, 21);"> </font><font style="color:rgb(15, 17, 21);">+</font><font style="color:rgb(15, 17, 21);"> </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">ip address</font>` |
| <font style="color:rgb(15, 17, 21);">封装 VLAN</font> | `<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">dot1q termination vid 10</font>` | <font style="color:rgb(15, 17, 21);">无需（VLANIF 自动关联）</font> |
| <font style="color:rgb(15, 17, 21);">ARP 广播</font> | <font style="color:rgb(15, 17, 21);">需手动</font><font style="color:rgb(15, 17, 21);"> </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">arp broadcast enable</font>` | <font style="color:rgb(15, 17, 21);">自动开启</font> |


---

## <font style="color:rgb(15, 17, 21);">五、最终配置检查清单</font>
+ <font style="color:rgb(15, 17, 21);">所有接口物理 UP（</font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">display interface brief</font>`<font style="color:rgb(15, 17, 21);">）</font>
+ <font style="color:rgb(15, 17, 21);">VLAN 创建并正确划分端口</font>
+ <font style="color:rgb(15, 17, 21);">Trunk 链路允许所有需要的 VLAN</font>
+ <font style="color:rgb(15, 17, 21);">VLANIF 接口 IP 配置正确，且状态 UP</font>
+ <font style="color:rgb(15, 17, 21);">OSPF 邻居 Full</font>
+ <font style="color:rgb(15, 17, 21);">路由表中有互访网段</font>
+ <font style="color:rgb(15, 17, 21);">DHCP 服务正常，PC 获取 IP</font>
+ <font style="color:rgb(15, 17, 21);">PC 能 ping 通网关、子接口、Loopback</font>
