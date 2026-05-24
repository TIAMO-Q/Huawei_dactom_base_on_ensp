<img src="https://cdn.nlark.com/yuque/0/2026/png/50508853/1776505588932-08a3e8fe-e95a-4870-a0b6-2629fb7c11b2.png" width="1288.6666666666667" title="" crop="0,0,1,1" id="uff010438" class="ne-image">

## <font style="color:rgb(15, 17, 21);">实验配置与知识点总结</font>
### <font style="color:rgb(15, 17, 21);">一、实验需求回顾</font>
+ <font style="color:rgb(15, 17, 21);">R1、R2、R3</font><font style="color:rgb(15, 17, 21);"> </font>**<font style="color:rgb(15, 17, 21);">只能配置一条静态路由</font>**<font style="color:rgb(15, 17, 21);">（R1 和 R2 实际上各配了多条，但题目可能指“核心路由”各一条，需结合拓扑理解）。</font>
+ <font style="color:rgb(15, 17, 21);">R4、R5、R6</font><font style="color:rgb(15, 17, 21);"> </font>**<font style="color:rgb(15, 17, 21);">只能配置 OSPF</font>**<font style="color:rgb(15, 17, 21);">。</font>
+ <font style="color:rgb(15, 17, 21);">实现全网通（所有设备能互访，包括 R1 的 Loopback 1.1.1.1、R6 的 Loopback 6.6.6.6 等）。</font>

---

### <font style="color:rgb(15, 17, 21);">二、拓扑结构与 IP 规划（根据命令反推）</font>
| <font style="color:rgb(15, 17, 21);">设备</font> | <font style="color:rgb(15, 17, 21);">接口</font> | <font style="color:rgb(15, 17, 21);">IP地址</font> | <font style="color:rgb(15, 17, 21);">对端设备</font> |
| --- | --- | --- | --- |
| <font style="color:rgb(15, 17, 21);">R1</font> | <font style="color:rgb(15, 17, 21);">GE0/0/1</font> | <font style="color:rgb(15, 17, 21);">192.168.255.2/30</font> | <font style="color:rgb(15, 17, 21);">R2 GE0/0/0 (192.168.255.1)</font> |
| <font style="color:rgb(15, 17, 21);">R1</font> | <font style="color:rgb(15, 17, 21);">Loopback1</font> | <font style="color:rgb(15, 17, 21);">1.1.1.1/32</font> | <font style="color:rgb(15, 17, 21);">—</font> |
| <font style="color:rgb(15, 17, 21);">R2</font> | <font style="color:rgb(15, 17, 21);">GE0/0/0</font> | <font style="color:rgb(15, 17, 21);">192.168.255.1/30</font> | <font style="color:rgb(15, 17, 21);">R1</font> |
| <font style="color:rgb(15, 17, 21);">R2</font> | <font style="color:rgb(15, 17, 21);">GE0/0/1</font> | <font style="color:rgb(15, 17, 21);">151.151.1.2/30</font> | <font style="color:rgb(15, 17, 21);">R3 GE0/0/0 (151.151.1.1)</font> |
| <font style="color:rgb(15, 17, 21);">R3</font> | <font style="color:rgb(15, 17, 21);">GE0/0/0</font> | <font style="color:rgb(15, 17, 21);">151.151.1.1/30</font> | <font style="color:rgb(15, 17, 21);">R2</font> |
| <font style="color:rgb(15, 17, 21);">R3</font> | <font style="color:rgb(15, 17, 21);">GE0/0/1</font> | <font style="color:rgb(15, 17, 21);">131.131.255.13/30</font> | <font style="color:rgb(15, 17, 21);">R4 GE0/0/0 (131.131.255.14)</font> |
| <font style="color:rgb(15, 17, 21);">R3</font> | <font style="color:rgb(15, 17, 21);">Serial4/0/0</font> | <font style="color:rgb(15, 17, 21);">131.131.255.6/30</font> | <font style="color:rgb(15, 17, 21);">R5 Serial4/0/0 (131.131.255.5)</font> |
| <font style="color:rgb(15, 17, 21);">R4</font> | <font style="color:rgb(15, 17, 21);">GE0/0/0</font> | <font style="color:rgb(15, 17, 21);">131.131.255.14/30</font> | <font style="color:rgb(15, 17, 21);">R3</font> |
| <font style="color:rgb(15, 17, 21);">R4</font> | <font style="color:rgb(15, 17, 21);">GE0/0/1</font> | <font style="color:rgb(15, 17, 21);">131.131.255.10/30</font> | <font style="color:rgb(15, 17, 21);">R5 GE0/0/0 (131.131.255.9)</font> |
| <font style="color:rgb(15, 17, 21);">R5</font> | <font style="color:rgb(15, 17, 21);">GE0/0/0</font> | <font style="color:rgb(15, 17, 21);">131.131.255.9/30</font> | <font style="color:rgb(15, 17, 21);">R4</font> |
| <font style="color:rgb(15, 17, 21);">R5</font> | <font style="color:rgb(15, 17, 21);">GE0/0/1</font> | <font style="color:rgb(15, 17, 21);">131.131.255.2/30</font> | <font style="color:rgb(15, 17, 21);">R6 GE0/0/0 (131.131.255.1)</font> |
| <font style="color:rgb(15, 17, 21);">R5</font> | <font style="color:rgb(15, 17, 21);">Serial4/0/0</font> | <font style="color:rgb(15, 17, 21);">131.131.255.5/30</font> | <font style="color:rgb(15, 17, 21);">R3</font> |
| <font style="color:rgb(15, 17, 21);">R6</font> | <font style="color:rgb(15, 17, 21);">GE0/0/0</font> | <font style="color:rgb(15, 17, 21);">131.131.255.1/30</font> | <font style="color:rgb(15, 17, 21);">R5</font> |
| <font style="color:rgb(15, 17, 21);">R6</font> | <font style="color:rgb(15, 17, 21);">Loopback0</font> | <font style="color:rgb(15, 17, 21);">6.6.6.6/32</font> | <font style="color:rgb(15, 17, 21);">—</font> |


---

### <font style="color:rgb(15, 17, 21);">三、配置命令分析（逐条解释）</font>
#### <font style="color:rgb(15, 17, 21);">R1 配置</font>
<font style="color:rgb(15, 17, 21);">bash</font>

```plain
sysname R1
interface GigabitEthernet0/0/1
 ip address 192.168.255.2 30
interface LoopBack1
 ip address 1.1.1.1 32
ip route-static 0.0.0.0 0 192.168.255.1
```

+ **<font style="color:rgb(15, 17, 21);">缺省路由</font>**<font style="color:rgb(15, 17, 21);">：指向 R2（192.168.255.1），使 R1 能够访问所有未知网络（包括 OSPF 域内的 6.6.6.6 等）。</font>
+ **<font style="color:rgb(15, 17, 21);">Loopback1</font>**<font style="color:rgb(15, 17, 21);">：模拟内网主机地址。</font>

#### <font style="color:rgb(15, 17, 21);">R2 配置</font>
<font style="color:rgb(15, 17, 21);">bash</font>

```plain
sysname R2
interface GigabitEthernet0/0/0
 ip address 192.168.255.1 30
interface GigabitEthernet0/0/1
 ip address 151.151.1.2 30
ip route-static 1.1.1.1 32 192.168.255.2
ip route-static 0.0.0.0 0 151.151.1.1
ip route-static 6.6.6.6 32 151.151.1.1
```

+ **<font style="color:rgb(15, 17, 21);">明细路由</font>**<font style="color:rgb(15, 17, 21);">：</font>
    - `<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">1.1.1.1</font>`<font style="color:rgb(15, 17, 21);"> </font><font style="color:rgb(15, 17, 21);">指向 R1（回程路由）。</font>
    - `<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">6.6.6.6</font>`<font style="color:rgb(15, 17, 21);"> </font><font style="color:rgb(15, 17, 21);">指向 R3（OSPF 域内的目标，但实际 R3 会通过 OSPF 学习到，静态为备份或辅助）。</font>
+ **<font style="color:rgb(15, 17, 21);">缺省路由</font>**<font style="color:rgb(15, 17, 21);">：指向 R3（151.151.1.1），使 R2 能够访问 OSPF 域。</font>

#### <font style="color:rgb(15, 17, 21);">R3 配置</font>
<font style="color:rgb(15, 17, 21);">bash</font>

```plain
sysname R3
interface GigabitEthernet0/0/0
 ip address 151.151.1.1 30
interface GigabitEthernet0/0/1
 ip address 131.131.255.13 30
interface Serial4/0/0
 ip address 131.131.255.6 30
ospf 1 router-id 3.3.3.3
 default-route-advertise
 area 0
  network 131.131.255.13 0.0.0.0
  network 131.131.255.6 0.0.0.0
ip route-static 0.0.0.0 0 151.151.1.2
```

+ **<font style="color:rgb(15, 17, 21);">OSPF 配置</font>**<font style="color:rgb(15, 17, 21);">：</font>
    - <font style="color:rgb(15, 17, 21);">宣告两个直连接口（</font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">131.131.255.13</font>`<font style="color:rgb(15, 17, 21);"> </font><font style="color:rgb(15, 17, 21);">和</font><font style="color:rgb(15, 17, 21);"> </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">131.131.255.6</font>`<font style="color:rgb(15, 17, 21);">），使它们参与 OSPF。</font>
    - `<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">default-route-advertise</font>`<font style="color:rgb(15, 17, 21);">：向 OSPF 域内发布默认路由（0.0.0.0/0），使 OSPF 域内的设备（R4/R5/R6）能通过 R3 访问外部（R2/R1）。</font>
+ **<font style="color:rgb(15, 17, 21);">静态缺省路由</font>**<font style="color:rgb(15, 17, 21);">：指向 R2（151.151.1.2），用于自身访问外部，也是 OSPF 默认路由的来源。</font>

#### <font style="color:rgb(15, 17, 21);">R4 配置</font>
<font style="color:rgb(15, 17, 21);">bash</font>

```plain
sysname R4
interface GigabitEthernet0/0/0
 ip address 131.131.255.14 30
interface GigabitEthernet0/0/1
 ip address 131.131.255.10 30
ospf 1 router-id 4.4.4.4
 area 0
  network 131.131.255.14 0.0.0.0
  network 131.131.255.10 0.0.0.0
```

+ <font style="color:rgb(15, 17, 21);">宣告两个接口，使它们成为 OSPF 的一部分。</font>

#### <font style="color:rgb(15, 17, 21);">R5 配置</font>
<font style="color:rgb(15, 17, 21);">bash</font>

```plain
sysname R5
interface GigabitEthernet0/0/0
 ip address 131.131.255.9 30
interface GigabitEthernet0/0/1
 ip address 131.131.255.2 30
interface Serial4/0/0
 ip address 131.131.255.5 30
ospf 1 router-id 5.5.5.5
 area 0
  network 131.131.255.2 0.0.0.0
  network 131.131.255.5 0.0.0.0
  network 131.131.255.9 0.0.0.0
```

+ <font style="color:rgb(15, 17, 21);">宣告三个接口，包括连接 R3 的串口和连接 R4、R6 的以太口。</font>

#### <font style="color:rgb(15, 17, 21);">R6 配置</font>
<font style="color:rgb(15, 17, 21);">bash</font>

```plain
sysname R6
interface GigabitEthernet0/0/0
 ip address 131.131.255.1 30
interface LoopBack0
 ip address 6.6.6.6 32
ospf 1 router-id 6.6.6.6
 area 0
  network 131.131.255.1 0.0.0.0
  network 6.6.6.6 0.0.0.0
```

+ <font style="color:rgb(15, 17, 21);">宣告物理接口和 Loopback，使 6.6.6.6 在 OSPF 中发布。</font>

---

### <font style="color:rgb(15, 17, 21);">四、路由传递与全网通实现分析</font>
1. **<font style="color:rgb(15, 17, 21);">OSPF 域内（R3-R4-R5-R6）</font>**<font style="color:rgb(15, 17, 21);">：</font>
    - <font style="color:rgb(15, 17, 21);">所有接口都在 Area 0，通过 OSPF 学习到彼此的路由。</font>
    - <font style="color:rgb(15, 17, 21);">R3 通过</font><font style="color:rgb(15, 17, 21);"> </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">default-route-advertise</font>`<font style="color:rgb(15, 17, 21);"> </font><font style="color:rgb(15, 17, 21);">向域内发布默认路由。</font>
    - <font style="color:rgb(15, 17, 21);">因此 R4/R5/R6 都能通过 R3 访问外部（R2/R1）。</font>
2. **<font style="color:rgb(15, 17, 21);">R3 与外部（R2/R1）</font>**<font style="color:rgb(15, 17, 21);">：</font>
    - <font style="color:rgb(15, 17, 21);">R3 有一条静态缺省路由指向 R2。</font>
    - <font style="color:rgb(15, 17, 21);">R2 有静态路由指向 1.1.1.1 和 6.6.6.6（实际上 6.6.6.6 可通过 OSPF 学习，但这里也写了静态，冗余）。</font>
    - <font style="color:rgb(15, 17, 21);">R2 也有缺省路由指向 R3，形成双向互通。</font>
3. **<font style="color:rgb(15, 17, 21);">R1 与外部</font>**<font style="color:rgb(15, 17, 21);">：</font>
    - <font style="color:rgb(15, 17, 21);">R1 只有一条缺省路由指向 R2，可到达任何网络。</font>
    - <font style="color:rgb(15, 17, 21);">R1 的 Loopback 1.1.1.1 需要被 R2 明确路由才能被 OSPF 域访问。</font>
4. **<font style="color:rgb(15, 17, 21);">全网通验证</font>**<font style="color:rgb(15, 17, 21);">：</font>
    - <font style="color:rgb(15, 17, 21);">R1 ping 6.6.6.6：R1 → R2 → R3 → R5 → R6（回程相同）。</font>
    - <font style="color:rgb(15, 17, 21);">R6 ping 1.1.1.1：R6 → R5 → R3 → R2 → R1（回程相同）。</font>

---

### <font style="color:rgb(15, 17, 21);">五、知识点总结</font>
#### <font style="color:rgb(15, 17, 21);">1. 静态路由</font>
+ **<font style="color:rgb(15, 17, 21);">缺省路由</font>**<font style="color:rgb(15, 17, 21);">：</font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">0.0.0.0 0</font>`<font style="color:rgb(15, 17, 21);"> </font><font style="color:rgb(15, 17, 21);">匹配所有目标，用于指定默认出口。</font>
+ **<font style="color:rgb(15, 17, 21);">明细路由</font>**<font style="color:rgb(15, 17, 21);">：指定具体目标网络，掩码长度越长越优先。</font>
+ **<font style="color:rgb(15, 17, 21);">浮动路由</font>**<font style="color:rgb(15, 17, 21);">：通过修改</font><font style="color:rgb(15, 17, 21);"> </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">preference</font>`<font style="color:rgb(15, 17, 21);"> </font><font style="color:rgb(15, 17, 21);">实现主备，本实验未使用。</font>
+ **<font style="color:rgb(15, 17, 21);">递归路由</font>**<font style="color:rgb(15, 17, 21);">：静态路由的下一跳可以是间接地址，需要路由表中有到达该地址的路由。</font>

#### <font style="color:rgb(15, 17, 21);">2. OSPF 基础</font>
+ **<font style="color:rgb(15, 17, 21);">Router ID</font>**<font style="color:rgb(15, 17, 21);">：唯一标识，需手动指定（推荐）或自动选举。</font>
+ **<font style="color:rgb(15, 17, 21);">区域（Area）</font>**<font style="color:rgb(15, 17, 21);">：Area 0 是骨干区域，所有非骨干必须直连 Area 0。</font>
+ **<font style="color:rgb(15, 17, 21);">Network 命令</font>**<font style="color:rgb(15, 17, 21);">：使用反掩码（wildcard mask）宣告接口，</font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">0.0.0.0</font>`<font style="color:rgb(15, 17, 21);"> </font><font style="color:rgb(15, 17, 21);">表示精确匹配该 IP。</font>
+ **<font style="color:rgb(15, 17, 21);">LSA 类型</font>**<font style="color:rgb(15, 17, 21);">：</font>
    - <font style="color:rgb(15, 17, 21);">Type 1 (Router LSA)：每台路由器产生，描述直连链路。</font>
    - <font style="color:rgb(15, 17, 21);">Type 2 (Network LSA)：DR 产生，描述广播型网络。</font>
    - <font style="color:rgb(15, 17, 21);">Type 3 (Network Summary LSA)：ABR 产生，描述区域间路由。</font>
    - <font style="color:rgb(15, 17, 21);">Type 5 (AS External LSA)：ASBR 产生，描述外部路由（如默认路由）。</font>

#### <font style="color:rgb(15, 17, 21);">3. 默认路由发布</font>
+ `**<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">default-route-advertise</font>**`<font style="color:rgb(15, 17, 21);">：在 ASBR 上配置，向 OSPF 域内发布缺省路由。前提是 ASBR 自身已有缺省路由（静态或动态）。</font>
+ `**<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">always</font>**`**<font style="color:rgb(15, 17, 21);"> </font>****<font style="color:rgb(15, 17, 21);">参数</font>**<font style="color:rgb(15, 17, 21);">：即使本地没有缺省路由也会发布（本实验未使用）。</font>

#### <font style="color:rgb(15, 17, 21);">4. 路由选路原则</font>
+ **<font style="color:rgb(15, 17, 21);">最长匹配优先</font>**<font style="color:rgb(15, 17, 21);">：掩码长度越大越优先。</font>
+ **<font style="color:rgb(15, 17, 21);">优先级（Preference）</font>**<font style="color:rgb(15, 17, 21);">：静态路由默认 60，OSPF 内部路由默认 10，外部路由默认 150。数值越小越优先。</font>
+ **<font style="color:rgb(15, 17, 21);">度量值（Cost）</font>**<font style="color:rgb(15, 17, 21);">：OSPF 使用 Cost 选路，默认参考带宽 100Mbps，Cost = 参考带宽 / 接口带宽。</font>

#### <font style="color:rgb(15, 17, 21);">5. 全网通设计思路</font>
+ <font style="color:rgb(15, 17, 21);">将网络划分为</font><font style="color:rgb(15, 17, 21);"> </font>**<font style="color:rgb(15, 17, 21);">OSPF 域</font>**<font style="color:rgb(15, 17, 21);">（内部）和</font><font style="color:rgb(15, 17, 21);"> </font>**<font style="color:rgb(15, 17, 21);">外部</font>**<font style="color:rgb(15, 17, 21);">（静态路由区域）。</font>
+ <font style="color:rgb(15, 17, 21);">在边界路由器（ASBR）上：</font>
    - <font style="color:rgb(15, 17, 21);">向 OSPF 域内注入默认路由（</font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">default-route-advertise</font>`<font style="color:rgb(15, 17, 21);">）。</font>
    - <font style="color:rgb(15, 17, 21);">在外部配置静态路由指向内部网段（或使用缺省路由）。</font>
+ <font style="color:rgb(15, 17, 21);">确保路由</font>**<font style="color:rgb(15, 17, 21);">双向可达</font>**<font style="color:rgb(15, 17, 21);">。</font>

#### <font style="color:rgb(15, 17, 21);">6. 实验中的特殊点</font>
+ **<font style="color:rgb(15, 17, 21);">R2 同时配置了静态缺省和明细</font>**<font style="color:rgb(15, 17, 21);">：明细用于精确回程，缺省用于未知网络。</font>
+ **<font style="color:rgb(15, 17, 21);">R3 同时运行 OSPF 和静态路由</font>**<font style="color:rgb(15, 17, 21);">：作为 ASBR。</font>
+ **<font style="color:rgb(15, 17, 21);">R5 有多个 OSPF 接口</font>**<font style="color:rgb(15, 17, 21);">：串口和以太口，注意 OSPF 网络类型不同（串口默认 P2P，以太口默认 Broadcast）。</font>

---

### <font style="color:rgb(15, 17, 21);">六、验证命令</font>
| <font style="color:rgb(15, 17, 21);">设备</font> | <font style="color:rgb(15, 17, 21);">命令</font> | <font style="color:rgb(15, 17, 21);">预期结果</font> |
| --- | --- | --- |
| <font style="color:rgb(15, 17, 21);">R1</font> | `<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">display ip routing-table</font>` | <font style="color:rgb(15, 17, 21);">缺省路由 0.0.0.0/0 via 192.168.255.1</font> |
| <font style="color:rgb(15, 17, 21);">R1</font> | `<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">ping 6.6.6.6</font>` | <font style="color:rgb(15, 17, 21);">通</font> |
| <font style="color:rgb(15, 17, 21);">R6</font> | `<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">display ip routing-table</font>` | <font style="color:rgb(15, 17, 21);">OSPF 默认路由 0.0.0.0/0 via 下一跳（如 131.131.255.2）</font> |
| <font style="color:rgb(15, 17, 21);">R6</font> | `<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">ping 1.1.1.1</font>` | <font style="color:rgb(15, 17, 21);">通</font> |
| <font style="color:rgb(15, 17, 21);">R3</font> | `<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">display ospf peer</font>` | <font style="color:rgb(15, 17, 21);">邻居状态 Full</font> |
| <font style="color:rgb(15, 17, 21);">R3</font> | `<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">display ospf lsdb</font>` | <font style="color:rgb(15, 17, 21);">查看 Type 5 LSA（默认路由）</font> |


---

### <font style="color:rgb(15, 17, 21);">七、常见问题与排错</font>
1. **<font style="color:rgb(15, 17, 21);">OSPF 邻居无法 Full</font>**<font style="color:rgb(15, 17, 21);">：检查接口 IP、区域 ID、认证（本实验未启用）、网络类型是否一致。</font>
2. **<font style="color:rgb(15, 17, 21);">默认路由未注入</font>**<font style="color:rgb(15, 17, 21);">：确认 R3 上</font><font style="color:rgb(15, 17, 21);"> </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">default-route-advertise</font>`<font style="color:rgb(15, 17, 21);"> </font><font style="color:rgb(15, 17, 21);">已配置，且 R3 自身有缺省路由（</font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">display ip routing-table</font>`<font style="color:rgb(15, 17, 21);">）。</font>
3. **<font style="color:rgb(15, 17, 21);">静态路由不生效</font>**<font style="color:rgb(15, 17, 21);">：检查下一跳是否可达，或配置错误（如掩码写错）。</font>
4. **<font style="color:rgb(15, 17, 21);">路由环路</font>**<font style="color:rgb(15, 17, 21);">：避免在多个边界同时注入默认路由，本实验只有一个 ASBR（R3），无环路风险。</font>

---

### <font style="color:rgb(15, 17, 21);">八、扩展思考</font>
+ <font style="color:rgb(15, 17, 21);">如果 R2 只有一条静态路由（题目要求），能否只配缺省路由？</font><font style="color:rgb(15, 17, 21);">  
</font><font style="color:rgb(15, 17, 21);">可以，但需确保 R1 的路由也能通过 R2 的缺省传递到 R3，且 R3 的默认路由注入后，R2 也能学习到 OSPF 域内的具体路由（静态缺省 + OSPF 动态路由混合）。</font>
+ <font style="color:rgb(15, 17, 21);">本实验中 R2 配了多条静态路由，其实可以精简为一条缺省路由指向 R3，再加一条去往 1.1.1.1 的明细（或让 R1 通告 OSPF，但 R1 不能运行 OSPF）。因此题目限制“只能一条静态”可能指“只能配置一条静态缺省”，但实际为了精确回程，有时需要多条。</font>
