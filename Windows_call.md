<img src="https://cdn.nlark.com/yuque/0/2026/png/50508853/1776600140885-5ddfdc3b-cba2-48d0-8be0-a59011e0e44e.png" width="1326" title="" crop="0,0,1,1" id="u7f2e5aed" class="ne-image">

## <font style="color:rgb(15, 17, 21);">PPPoE 实验完整解析（结论先行）</font>
### <font style="color:rgb(15, 17, 21);">实验目标</font>
+ **<font style="color:rgb(15, 17, 21);">R2 作为 PPPoE 服务器</font>**<font style="color:rgb(15, 17, 21);">，为拨入的客户端分配 IP 地址（</font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">20.20.2.0/24</font>`<font style="color:rgb(15, 17, 21);"> </font><font style="color:rgb(15, 17, 21);">网段），使用 CHAP 认证。</font>
+ **<font style="color:rgb(15, 17, 21);">R1 作为 PPPoE 客户端</font>**<font style="color:rgb(15, 17, 21);">，通过 Dialer 接口拨号，获取 IP 后做 NAT，让内网 PC 上网。</font>
+ **<font style="color:rgb(15, 17, 21);">LSW1 作为二层交换机</font>**<font style="color:rgb(15, 17, 21);">，所有接口划分到 VLAN 1，保证 R1 和 R2 在同一个广播域（PPPoE 发现阶段需要广播）。</font>
+ **<font style="color:rgb(15, 17, 21);">LSW2 作为三层交换机</font>**<font style="color:rgb(15, 17, 21);">，划分 VLAN 10、20、13，为 PC1、PC2 提供 DHCP 服务，并连接 R1 的内网接口。</font>

---

## <font style="color:rgb(15, 17, 21);">一、PPPoE 服务端（R2）配置解析</font>
### <font style="color:rgb(15, 17, 21);">1.1 AAA 认证配置</font>
<font style="color:rgb(15, 17, 21);">bash</font>

```plain
[R2] aaa
[R2-aaa] local-user long password cipher 123
[R2-aaa] local-user long service-type ppp
```

**<font style="color:rgb(15, 17, 21);">命令解析</font>**<font style="color:rgb(15, 17, 21);">：</font>

+ `<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">local-user long password cipher 123</font>`<font style="color:rgb(15, 17, 21);">：创建本地用户</font><font style="color:rgb(15, 17, 21);"> </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">long</font>`<font style="color:rgb(15, 17, 21);">，密码密文存储为</font><font style="color:rgb(15, 17, 21);"> </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">123</font>`<font style="color:rgb(15, 17, 21);">。</font>
+ `<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">service-type ppp</font>`<font style="color:rgb(15, 17, 21);">：指定该用户用于 PPP 认证（必须配置，否则认证失败）。</font>

### <font style="color:rgb(15, 17, 21);">1.2 地址池与虚拟模板</font>
<font style="color:rgb(15, 17, 21);">bash</font>

```plain
[R2] ip pool pppoe
[R2-ip-pool-pppoe] network 20.20.2.0 mask 24
[R2-ip-pool-pppoe] gateway-list 20.20.2.254
[R2] interface Virtual-Template1
[R2-Virtual-Template1] ppp authentication-mode chap
[R2-Virtual-Template1] ip address 20.20.2.254 24
[R2-Virtual-Template1] remote address pool pppoe
[R2-Virtual-Template1] ppp ipcp dns 114.114.114.114
[R2-Virtual-Template1] ppp ipcp default-route
```

**<font style="color:rgb(15, 17, 21);">命令解析</font>**<font style="color:rgb(15, 17, 21);">：</font>

+ `<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">ip pool pppoe</font>`<font style="color:rgb(15, 17, 21);">：定义地址池，用于分配给客户端。</font>
+ `<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">gateway-list</font>`<font style="color:rgb(15, 17, 21);">：指定客户端获取的网关地址（即服务器自身的 IP）。</font>
+ `<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">Virtual-Template1</font>`<font style="color:rgb(15, 17, 21);">：虚拟模板，配置 PPP 参数（认证、IP、地址池、DNS 等）。</font>
+ `<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">ppp authentication-mode chap</font>`<font style="color:rgb(15, 17, 21);">：要求客户端使用 CHAP 认证。</font>
+ `<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">remote address pool pppoe</font>`<font style="color:rgb(15, 17, 21);">：从地址池</font><font style="color:rgb(15, 17, 21);"> </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">pppoe</font>`<font style="color:rgb(15, 17, 21);"> </font><font style="color:rgb(15, 17, 21);">中为客户端分配 IP。</font>
+ `<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">ppp ipcp dns</font>`<font style="color:rgb(15, 17, 21);">：通过 IPCP 下发给客户端的 DNS 服务器地址。</font>
+ `<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">ppp ipcp default-route</font>`<font style="color:rgb(15, 17, 21);">：要求客户端自动添加缺省路由（华为设备上通常不需要，由客户端自己配置）。</font>

### <font style="color:rgb(15, 17, 21);">1.3 物理接口绑定</font>
<font style="color:rgb(15, 17, 21);">bash</font>

```plain
[R2] interface GigabitEthernet0/0/0
[R2-GigabitEthernet0/0/0] ip address 192.168.100.91 24
[R2-GigabitEthernet0/0/0] pppoe-server bind virtual-template 1
```

**<font style="color:rgb(15, 17, 21);">命令解析</font>**<font style="color:rgb(15, 17, 21);">：</font>

+ `<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">pppoe-server bind virtual-template 1</font>`<font style="color:rgb(15, 17, 21);">：将物理接口与虚拟模板绑定，使该接口提供 PPPoE 服务。此接口通常不需要配置 IP 地址（但配了也无妨），因为 PPPoE 基于以太网 MAC 地址，不依赖 IP。</font>

---

## <font style="color:rgb(15, 17, 21);">二、二层交换机 LSW1 配置</font>
<font style="color:rgb(15, 17, 21);">bash</font>

```plain
[SW1] vlan 1
[SW1] interface Ethernet0/0/1
[SW1-Ethernet0/0/1] port link-type access
[SW1-Ethernet0/0/1] port default vlan 1
[SW1-Ethernet0/0/2] ...  # 同理配置所有端口
```

**<font style="color:rgb(15, 17, 21);">知识点</font>**<font style="color:rgb(15, 17, 21);">：</font>

+ <font style="color:rgb(15, 17, 21);">所有端口划入 VLAN 1，保证 R1（</font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">GE0/0/0</font>`<font style="color:rgb(15, 17, 21);">）和 R2（</font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">GE0/0/0</font>`<font style="color:rgb(15, 17, 21);">）在同一个广播域，使 PPPoE 发现阶段的广播报文（PADI）能够到达服务器。</font>
+ <font style="color:rgb(15, 17, 21);">如果使用 eNSP 的 Cloud 连接真实主机，需要确保 Cloud 的绑定端口也属于 VLAN 1。</font>

---

## <font style="color:rgb(15, 17, 21);">三、PPPoE 客户端（R1）配置解析</font>
### <font style="color:rgb(15, 17, 21);">3.1 AAA 配置（可选，客户端通常不需要）</font>
<font style="color:rgb(15, 17, 21);">bash</font>

```plain
[R1] aaa
[R1-aaa] local-user long password cipher 123
[R1-aaa] local-user long service-type ppp
```

**<font style="color:rgb(15, 17, 21);">说明</font>**<font style="color:rgb(15, 17, 21);">：客户端也可以配置本地用户，但本实验中 R1 作为客户端，</font>**<font style="color:rgb(15, 17, 21);">不需要 AAA 认证别人</font>**<font style="color:rgb(15, 17, 21);">，实际上可以省略。保留不影响。</font>

### <font style="color:rgb(15, 17, 21);">3.2 Dialer 接口配置</font>
<font style="color:rgb(15, 17, 21);">bash</font>

```plain
[R1] interface Dialer1
[R1-Dialer1] dialer user long
[R1-Dialer1] dialer bundle 1
[R1-Dialer1] ppp chap user long
[R1-Dialer1] ppp chap password cipher 123
[R1-Dialer1] ip address ppp-negotiate
[R1-Dialer1] ppp ipcp default-route
[R1-Dialer1] nat outbound 2000
```

**<font style="color:rgb(15, 17, 21);">命令解析</font>**<font style="color:rgb(15, 17, 21);">：</font>

+ `<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">dialer user long</font>`<font style="color:rgb(15, 17, 21);">：指定拨号用户名（用于 CHAP 认证，可选但建议配置）。</font>
+ `<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">dialer bundle 1</font>`<font style="color:rgb(15, 17, 21);">：将 Dialer 接口与物理接口绑定（通过 bundle 编号关联）。</font>
+ `<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">ppp chap user long</font>`<font style="color:rgb(15, 17, 21);">：CHAP 认证时发送的用户名。</font>
+ `<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">ppp chap password cipher 123</font>`<font style="color:rgb(15, 17, 21);">：CHAP 认证的密码。</font>
+ `<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">ip address ppp-negotiate</font>`<font style="color:rgb(15, 17, 21);">：通过 IPCP 从服务器获取 IP 地址。</font>
+ `<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">ppp ipcp default-route</font>`<font style="color:rgb(15, 17, 21);">：自动添加缺省路由指向服务器。</font>
+ `<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">nat outbound 2000</font>`<font style="color:rgb(15, 17, 21);">：对匹配 ACL 2000 的流量进行 Easy IP 转换（内网上网）。</font>

### <font style="color:rgb(15, 17, 21);">3.3 物理接口绑定</font>
<font style="color:rgb(15, 17, 21);">bash</font>

```plain
[R1] interface GigabitEthernet0/0/0
[R1-GigabitEthernet0/0/0] ip address 192.168.100.25 24
[R1-GigabitEthernet0/0/0] pppoe-client dial-bundle-number 1
```

**<font style="color:rgb(15, 17, 21);">注意</font>**<font style="color:rgb(15, 17, 21);">：</font>

+ <font style="color:rgb(15, 17, 21);">物理接口配置 IP 地址不是必须的（PPPoE 客户端可以没有 IP），但有时用于调试或管理。</font>
+ `<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">pppoe-client dial-bundle-number 1</font>`<font style="color:rgb(15, 17, 21);">：指定该物理接口使用 Dialer bundle 1 进行 PPPoE 拨号。</font>

### <font style="color:rgb(15, 17, 21);">3.4 内网接口与静态路由</font>
<font style="color:rgb(15, 17, 21);">bash</font>

```plain
[R1] interface GigabitEthernet0/0/1
[R1-GigabitEthernet0/0/1] ip address 192.168.13.1 24
[R1] ip route-static 192.168.10.0 24 192.168.13.2
[R1] ip route-static 192.168.20.0 24 192.168.13.2
```

+ `<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">GE0/0/1</font>`<font style="color:rgb(15, 17, 21);"> </font><font style="color:rgb(15, 17, 21);">连接 LSW2（三层交换机），作为内网网关。</font>
+ <font style="color:rgb(15, 17, 21);">静态路由指向 LSW2 的 VLANIF 13 接口（</font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">192.168.13.2</font>`<font style="color:rgb(15, 17, 21);">），使得 R1 知道如何到达内网 VLAN 10 和 VLAN 20。</font>

---

## <font style="color:rgb(15, 17, 21);">四、三层交换机 LSW2 配置（内网）</font>
### <font style="color:rgb(15, 17, 21);">4.1 VLAN 与 DHCP</font>
<font style="color:rgb(15, 17, 21);">bash</font>

```plain
[LSW2] vlan batch 10 20 13
[LSW2] dhcp enable
[LSW2] interface Ethernet0/0/2
[LSW2-Ethernet0/0/2] port link-type access
[LSW2-Ethernet0/0/2] port default vlan 10
[LSW2] interface Ethernet0/0/3
[LSW2-Ethernet0/0/3] port link-type access
[LSW2-Ethernet0/0/3] port default vlan 20
[LSW2] interface Ethernet0/0/1
[LSW2-Ethernet0/0/1] port link-type access
[LSW2-Ethernet0/0/1] port default vlan 13
```

+ <font style="color:rgb(15, 17, 21);">PC1 接 E0/0/2（VLAN 10），PC2 接 E0/0/3（VLAN 20），上联 R1 的接口 E0/0/1 属于 VLAN 13。</font>

### <font style="color:rgb(15, 17, 21);">4.2 VLANIF 接口与 DHCP 服务</font>
<font style="color:rgb(15, 17, 21);">bash</font>

```plain
[LSW2] interface Vlanif10
[LSW2-Vlanif10] ip address 192.168.10.1 24
[LSW2-Vlanif10] dhcp select interface
[LSW2] interface Vlanif20
[LSW2-Vlanif20] ip address 192.168.20.1 24
[LSW2-Vlanif20] dhcp select interface
[LSW2] interface Vlanif13
[LSW2-Vlanif13] ip address 192.168.13.2 24
[LSW2] ip route-static 0.0.0.0 0 192.168.13.1
```

+ <font style="color:rgb(15, 17, 21);">VLANIF 10/20 作为 PC 的网关，并开启 DHCP 接口地址池。</font>
+ <font style="color:rgb(15, 17, 21);">VLANIF 13 与 R1 的</font><font style="color:rgb(15, 17, 21);"> </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">192.168.13.1</font>`<font style="color:rgb(15, 17, 21);"> </font><font style="color:rgb(15, 17, 21);">三层互通。</font>
+ <font style="color:rgb(15, 17, 21);">缺省路由指向 R1，使内网流量能够出去（经过 R1 的 NAT）。</font>

---

## <font style="color:rgb(15, 17, 21);">五、PPPoE 建立过程与抓包细节</font>
### <font style="color:rgb(15, 17, 21);">5.1 PPPoE 发现阶段（广播）</font>
| <font style="color:rgb(15, 17, 21);">报文</font> | <font style="color:rgb(15, 17, 21);">方向</font> | <font style="color:rgb(15, 17, 21);">作用</font> |
| --- | --- | --- |
| **<font style="color:rgb(15, 17, 21);">PADI</font>** | <font style="color:rgb(15, 17, 21);">Client → Server</font> | <font style="color:rgb(15, 17, 21);">客户端广播寻找服务器（目的 MAC 全 F）</font> |
| **<font style="color:rgb(15, 17, 21);">PADO</font>** | <font style="color:rgb(15, 17, 21);">Server → Client</font> | <font style="color:rgb(15, 17, 21);">服务器单播回应，告知自己的 MAC 和名称</font> |
| **<font style="color:rgb(15, 17, 21);">PADR</font>** | <font style="color:rgb(15, 17, 21);">Client → Server</font> | <font style="color:rgb(15, 17, 21);">客户端单播请求建立会话</font> |
| **<font style="color:rgb(15, 17, 21);">PADS</font>** | <font style="color:rgb(15, 17, 21);">Server → Client</font> | <font style="color:rgb(15, 17, 21);">服务器分配会话 ID，确认建立</font> |


**<font style="color:rgb(15, 17, 21);">抓包位置</font>**<font style="color:rgb(15, 17, 21);">：在 LSW1 上镜像端口，或直接在 R1 的</font><font style="color:rgb(15, 17, 21);"> </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">GE0/0/0</font>`<font style="color:rgb(15, 17, 21);"> </font><font style="color:rgb(15, 17, 21);">抓包，过滤</font><font style="color:rgb(15, 17, 21);"> </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">pppoed</font>`<font style="color:rgb(15, 17, 21);">（发现阶段）和</font><font style="color:rgb(15, 17, 21);"> </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">pppoes</font>`<font style="color:rgb(15, 17, 21);">（会话阶段）。</font>

### <font style="color:rgb(15, 17, 21);">5.2 PPP 会话阶段（LCP → CHAP → IPCP）</font>
1. **<font style="color:rgb(15, 17, 21);">LCP 协商</font>**<font style="color:rgb(15, 17, 21);">：配置选项（MRU、认证协议 CHAP、魔术字等）。</font>
2. **<font style="color:rgb(15, 17, 21);">CHAP 认证</font>**<font style="color:rgb(15, 17, 21);">：</font>
    - <font style="color:rgb(15, 17, 21);">服务器发送 Challenge（随机数 + ID）。</font>
    - <font style="color:rgb(15, 17, 21);">客户端用密码 + Challenge 计算 MD5，发送 Response。</font>
    - <font style="color:rgb(15, 17, 21);">服务器验证成功，发送 Success。</font>
3. **<font style="color:rgb(15, 17, 21);">IPCP 协商</font>**<font style="color:rgb(15, 17, 21);">：客户端请求 IP，服务器分配</font><font style="color:rgb(15, 17, 21);"> </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">20.20.2.x</font>`<font style="color:rgb(15, 17, 21);">，同时下发 DNS、缺省路由等。</font>

---

## <font style="color:rgb(15, 17, 21);">六、验证命令</font>
| <font style="color:rgb(15, 17, 21);">设备</font> | <font style="color:rgb(15, 17, 21);">命令</font> | <font style="color:rgb(15, 17, 21);">预期结果</font> |
| --- | --- | --- |
| <font style="color:rgb(15, 17, 21);">R1</font> | `<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">display pppoe-client session summary</font>` | <font style="color:rgb(15, 17, 21);">状态为</font><font style="color:rgb(15, 17, 21);"> </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">UP</font>`<br/><font style="color:rgb(15, 17, 21);">，有 Session ID</font> |
| <font style="color:rgb(15, 17, 21);">R1</font> | `<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">display interface Dialer1</font>` | <font style="color:rgb(15, 17, 21);">显示获取到的 IP（如</font><font style="color:rgb(15, 17, 21);"> </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">20.20.2.2/32</font>`<br/><font style="color:rgb(15, 17, 21);">）</font> |
| <font style="color:rgb(15, 17, 21);">R1</font> | `<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">display ip routing-table</font>` | <font style="color:rgb(15, 17, 21);">缺省路由指向 Dialer1</font> |
| <font style="color:rgb(15, 17, 21);">R1</font> | `<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">ping 20.20.2.254</font>` | <font style="color:rgb(15, 17, 21);">通（服务器网关）</font> |
| <font style="color:rgb(15, 17, 21);">R2</font> | `<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">display pppoe-server session all</font>` | <font style="color:rgb(15, 17, 21);">显示客户端的 MAC 和分配的 IP</font> |
| <font style="color:rgb(15, 17, 21);">LSW2</font> | `<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">display ip interface brief</font>` | <font style="color:rgb(15, 17, 21);">VLANIF 接口 UP，DHCP 正常</font> |
| <font style="color:rgb(15, 17, 21);">PC1</font> | `<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">ipconfig</font>` | <font style="color:rgb(15, 17, 21);">获取</font><font style="color:rgb(15, 17, 21);"> </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">192.168.10.x</font>`<br/><font style="color:rgb(15, 17, 21);">，网关</font><font style="color:rgb(15, 17, 21);"> </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">192.168.10.1</font>` |
| <font style="color:rgb(15, 17, 21);">PC1</font> | `<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">ping 8.8.8.8</font>` | <font style="color:rgb(15, 17, 21);">通（通过 R1 的 NAT）</font> |


---

## <font style="color:rgb(15, 17, 21);">七、常见排错</font>
| <font style="color:rgb(15, 17, 21);">现象</font> | <font style="color:rgb(15, 17, 21);">可能原因</font> | <font style="color:rgb(15, 17, 21);">解决方法</font> |
| --- | --- | --- |
| <font style="color:rgb(15, 17, 21);">R1 拨号失败，</font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">Dialer1</font>`<br/><font style="color:rgb(15, 17, 21);"> </font><font style="color:rgb(15, 17, 21);">无 IP</font> | <font style="color:rgb(15, 17, 21);">二层广播不通（VLAN 不一致、交换机端口 down）</font> | <font style="color:rgb(15, 17, 21);">检查 LSW1 端口状态，确保 R1 和 R2 在同一 VLAN 且能互 ping 直连 IP</font> |
| <font style="color:rgb(15, 17, 21);">认证失败</font> | <font style="color:rgb(15, 17, 21);">用户名/密码错误，或</font><font style="color:rgb(15, 17, 21);"> </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">service-type</font>`<br/><font style="color:rgb(15, 17, 21);"> </font><font style="color:rgb(15, 17, 21);">缺失</font> | <font style="color:rgb(15, 17, 21);">核对 R2 的 AAA 用户配置，确保</font><font style="color:rgb(15, 17, 21);"> </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">service-type ppp</font>` |
| <font style="color:rgb(15, 17, 21);">R1 获取 IP 但无法 ping 通服务器网关</font> | <font style="color:rgb(15, 17, 21);">服务器上未配置</font><font style="color:rgb(15, 17, 21);"> </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">ppp ipcp default-route</font>`<br/><font style="color:rgb(15, 17, 21);"> </font><font style="color:rgb(15, 17, 21);">或客户端未获取到路由</font> | <font style="color:rgb(15, 17, 21);">检查 R2 的 Virtual-Template 配置，R1 上手动添加路由测试</font> |
| <font style="color:rgb(15, 17, 21);">内网 PC 无法上网</font> | <font style="color:rgb(15, 17, 21);">R1 上 NAT 未配置或缺省路由未指向 Dialer1</font> | <font style="color:rgb(15, 17, 21);">确认</font><font style="color:rgb(15, 17, 21);"> </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">nat outbound 2000</font>`<br/><font style="color:rgb(15, 17, 21);"> </font><font style="color:rgb(15, 17, 21);">和</font><font style="color:rgb(15, 17, 21);"> </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">ppp ipcp default-route</font>` |


---

## <font style="color:rgb(15, 17, 21);">八、知识点拓展</font>
### <font style="color:rgb(15, 17, 21);">8.1</font><font style="color:rgb(15, 17, 21);"> </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">dialer bundle</font>`<font style="color:rgb(15, 17, 21);"> </font><font style="color:rgb(15, 17, 21);">与</font><font style="color:rgb(15, 17, 21);"> </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">pppoe-client dial-bundle-number</font>`<font style="color:rgb(15, 17, 21);"> </font><font style="color:rgb(15, 17, 21);">的对应关系</font>
+ `<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">dialer bundle 1</font>`<font style="color:rgb(15, 17, 21);"> </font><font style="color:rgb(15, 17, 21);">在 Dialer 接口下定义 bundle 编号。</font>
+ `<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">pppoe-client dial-bundle-number 1</font>`<font style="color:rgb(15, 17, 21);"> </font><font style="color:rgb(15, 17, 21);">在物理接口下引用同一个 bundle 编号，完成绑定。</font>
+ <font style="color:rgb(15, 17, 21);">一个 Dialer 接口可以绑定多个物理接口（实现多链路 PPP，MP）。</font>

### <font style="color:rgb(15, 17, 21);">8.2</font><font style="color:rgb(15, 17, 21);"> </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">dialer user</font>`<font style="color:rgb(15, 17, 21);"> </font><font style="color:rgb(15, 17, 21);">的作用</font>
+ <font style="color:rgb(15, 17, 21);">在 PPPoE 客户端中，</font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">dialer user</font>`<font style="color:rgb(15, 17, 21);"> </font><font style="color:rgb(15, 17, 21);">用于指定对端服务器用户名，用于认证和区分不同会话。</font>**<font style="color:rgb(15, 17, 21);">必须配置</font>**<font style="color:rgb(15, 17, 21);">，否则 Dialer 接口无法正常建立连接。</font>

### <font style="color:rgb(15, 17, 21);">8.3 为什么物理接口还要配 IP？</font>
+ <font style="color:rgb(15, 17, 21);">物理接口的 IP（如</font><font style="color:rgb(15, 17, 21);"> </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">192.168.100.25</font>`<font style="color:rgb(15, 17, 21);">）与 PPPoE 无关，仅用于管理或测试。如果不配置，不影响 PPPoE 拨号。</font>

---

## <font style="color:rgb(15, 17, 21);">九、最终配置检查清单</font>
+ <font style="color:rgb(15, 17, 21);">LSW1 所有端口 Access VLAN 1，且状态 UP。</font>
+ <font style="color:rgb(15, 17, 21);">R2 上</font><font style="color:rgb(15, 17, 21);"> </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">pppoe-server bind virtual-template 1</font>`<font style="color:rgb(15, 17, 21);"> </font><font style="color:rgb(15, 17, 21);">已配置。</font>
+ <font style="color:rgb(15, 17, 21);">R2 的 AAA 用户</font><font style="color:rgb(15, 17, 21);"> </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">long</font>`<font style="color:rgb(15, 17, 21);"> </font><font style="color:rgb(15, 17, 21);">有</font><font style="color:rgb(15, 17, 21);"> </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">service-type ppp</font>`<font style="color:rgb(15, 17, 21);">。</font>
+ <font style="color:rgb(15, 17, 21);">R1 的 Dialer1 配置了</font><font style="color:rgb(15, 17, 21);"> </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">dialer user</font>`<font style="color:rgb(15, 17, 21);">、</font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">dialer bundle</font>`<font style="color:rgb(15, 17, 21);">、</font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">ppp chap user/password</font>`<font style="color:rgb(15, 17, 21);">。</font>
+ <font style="color:rgb(15, 17, 21);">R1 的物理接口</font><font style="color:rgb(15, 17, 21);"> </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">pppoe-client dial-bundle-number 1</font>`<font style="color:rgb(15, 17, 21);"> </font><font style="color:rgb(15, 17, 21);">与 Dialer bundle 一致。</font>
+ <font style="color:rgb(15, 17, 21);">R1 有缺省路由指向 Dialer1（自动或手动）。</font>
+ <font style="color:rgb(15, 17, 21);">LSW2 有缺省路由指向 R1（</font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">192.168.13.1</font>`<font style="color:rgb(15, 17, 21);">）。</font>
+ <font style="color:rgb(15, 17, 21);">PC1 能 ping 通 </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">192.168.10.1</font>`<font style="color:rgb(15, 17, 21);"> 和 </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">8.8.8.8</font>`<font style="color:rgb(15, 17, 21);">。</font>
