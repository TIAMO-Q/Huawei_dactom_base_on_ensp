<img src="https://cdn.nlark.com/yuque/0/2026/png/50508853/1776583283091-41b95a1e-a694-4285-9556-dabc4a2958c0.png" width="1248" title="" crop="0,0,1,1" id="u9c1cbd28" class="ne-image">

## <font style="color:rgb(15, 17, 21);">一、整体实验目的与数据流</font>
+ **<font style="color:rgb(15, 17, 21);">内网</font>**<font style="color:rgb(15, 17, 21);">：</font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">192.168.1.0/24</font>`<font style="color:rgb(15, 17, 21);">，PC1、PC2、Client1 通过 R1 的 DHCP 接口地址池动态获取 IP（无固定绑定）。</font>
+ **<font style="color:rgb(15, 17, 21);">R1</font>**<font style="color:rgb(15, 17, 21);">：作为出口路由器，同时承担</font><font style="color:rgb(15, 17, 21);"> </font>**<font style="color:rgb(15, 17, 21);">DHCP 服务器</font>**<font style="color:rgb(15, 17, 21);">、</font>**<font style="color:rgb(15, 17, 21);">NAT（Easy IP + NAT Server）</font>**<font style="color:rgb(15, 17, 21);">、</font>**<font style="color:rgb(15, 17, 21);">ACL 过滤</font>**<font style="color:rgb(15, 17, 21);">、</font>**<font style="color:rgb(15, 17, 21);">DNS 代理</font>**<font style="color:rgb(15, 17, 21);">。</font>
+ **<font style="color:rgb(15, 17, 21);">外网</font>**<font style="color:rgb(15, 17, 21);">：</font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">12.0.0.0/24</font>`<font style="color:rgb(15, 17, 21);">（R1与ISP互联），</font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">2.0.0.0/24</font>`<font style="color:rgb(15, 17, 21);">（外网服务器所在网段），ISP 作为纯转发设备（无静态路由，仅靠直连路由）。</font>
+ **<font style="color:rgb(15, 17, 21);">外网服务器</font>**<font style="color:rgb(15, 17, 21);">：</font>
    - <font style="color:rgb(15, 17, 21);">淘宝（</font>[<font style="color:rgb(57, 100, 254);">www.taobao.com</font>](https://www.taobao.com/)<font style="color:rgb(15, 17, 21);">）：</font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">2.0.0.50</font>`
    - <font style="color:rgb(15, 17, 21);">电影 FTP（</font>[<font style="color:rgb(57, 100, 254);">ftp.movie.com</font>](https://ftp.movie.com/)<font style="color:rgb(15, 17, 21);">）：</font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">2.0.0.20</font>`
    - <font style="color:rgb(15, 17, 21);">DNS 服务器：</font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">2.0.0.66</font>`
+ **<font style="color:rgb(15, 17, 21);">内网服务器</font>**<font style="color:rgb(15, 17, 21);">：京东（</font>[<font style="color:rgb(57, 100, 254);">www.jd.com</font>](https://www.jd.com/)<font style="color:rgb(15, 17, 21);">）位于</font><font style="color:rgb(15, 17, 21);"> </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">192.168.1.100</font>`<font style="color:rgb(15, 17, 21);">，通过 NAT Server 对外发布。</font>

---

## <font style="color:rgb(15, 17, 21);">二、DHCP 接口地址池（重点：动态分配，无静态绑定）</font>
### <font style="color:rgb(15, 17, 21);">结论</font>
+ <font style="color:rgb(15, 17, 21);">R1 的</font><font style="color:rgb(15, 17, 21);"> </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">GigabitEthernet0/0/0</font>`<font style="color:rgb(15, 17, 21);"> </font><font style="color:rgb(15, 17, 21);">接口 IP 为</font><font style="color:rgb(15, 17, 21);"> </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">192.168.1.1/24</font>`<font style="color:rgb(15, 17, 21);">，启用</font><font style="color:rgb(15, 17, 21);"> </font>**<font style="color:rgb(15, 17, 21);">接口地址池模式</font>**<font style="color:rgb(15, 17, 21);">（</font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">dhcp select interface</font>`<font style="color:rgb(15, 17, 21);">），自动使用接口所在网段为内网主机分配 IP。</font>
+ <font style="color:rgb(15, 17, 21);">地址池范围：</font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">192.168.1.2</font>`<font style="color:rgb(15, 17, 21);"> </font><font style="color:rgb(15, 17, 21);">–</font><font style="color:rgb(15, 17, 21);"> </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">192.168.1.254</font>`<font style="color:rgb(15, 17, 21);">（网关</font><font style="color:rgb(15, 17, 21);"> </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">192.168.1.1</font>`<font style="color:rgb(15, 17, 21);"> </font><font style="color:rgb(15, 17, 21);">被排除）。</font>
+ **<font style="color:rgb(15, 17, 21);">没有静态绑定</font>**<font style="color:rgb(15, 17, 21);">，因此 PC1、PC2、Client1 每次可能获得不同 IP。</font>

### <font style="color:rgb(15, 17, 21);">抓包细节</font>
1. <font style="color:rgb(15, 17, 21);">在 R1 的 GE0/0/0 接口抓包，过滤</font><font style="color:rgb(15, 17, 21);"> </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">bootp</font>`<font style="color:rgb(15, 17, 21);"> </font><font style="color:rgb(15, 17, 21);">或</font><font style="color:rgb(15, 17, 21);"> </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">udp.port==67</font>`<font style="color:rgb(15, 17, 21);">。</font>
2. <font style="color:rgb(15, 17, 21);">当 PC 启动时，会发送</font><font style="color:rgb(15, 17, 21);"> </font>**<font style="color:rgb(15, 17, 21);">DHCP Discover</font>**<font style="color:rgb(15, 17, 21);">（广播，源IP 0.0.0.0，目的IP 255.255.255.255）。</font>
3. <font style="color:rgb(15, 17, 21);">R1 收到后，回复</font><font style="color:rgb(15, 17, 21);"> </font>**<font style="color:rgb(15, 17, 21);">DHCP Offer</font>**<font style="color:rgb(15, 17, 21);">（单播或广播），其中包含：</font>
    - `<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">Your (client) IP address</font>`<font style="color:rgb(15, 17, 21);">：例如</font><font style="color:rgb(15, 17, 21);"> </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">192.168.1.100</font>`
    - `<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">Subnet Mask</font>`<font style="color:rgb(15, 17, 21);">：</font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">255.255.255.0</font>`
    - `<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">Router</font>`<font style="color:rgb(15, 17, 21);">（网关）：</font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">192.168.1.1</font>`
    - `<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">DNS Server</font>`<font style="color:rgb(15, 17, 21);">：</font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">192.168.1.1</font>`<font style="color:rgb(15, 17, 21);">（因为</font><font style="color:rgb(15, 17, 21);"> </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">dns-list 192.168.1.1</font>`<font style="color:rgb(15, 17, 21);">）</font>

## <font style="color:rgb(15, 17, 21);">一、DHCP：如何让 PC 自动获取 IP 并正确指向 DNS 代理</font>
### <font style="color:rgb(15, 17, 21);">1. 全局地址池 vs 接口地址池</font>
| <font style="color:rgb(15, 17, 21);">模式</font> | <font style="color:rgb(15, 17, 21);">命令</font> | <font style="color:rgb(15, 17, 21);">特点</font> |
| --- | --- | --- |
| <font style="color:rgb(15, 17, 21);">接口地址池</font> | `<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">dhcp select interface</font>` | <font style="color:rgb(15, 17, 21);">自动使用接口 IP 所在网段，无需额外配置</font> |
| <font style="color:rgb(15, 17, 21);">全局地址池</font> | `<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">dhcp select global</font>`<br/><font style="color:rgb(15, 17, 21);"> </font><font style="color:rgb(15, 17, 21);">+</font><font style="color:rgb(15, 17, 21);"> </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">ip pool</font>` | <font style="color:rgb(15, 17, 21);">可精细控制网关、DNS、租期、排除地址、静态绑定</font> |


<font style="color:rgb(15, 17, 21);">你的实验中，内网只有一个网段</font><font style="color:rgb(15, 17, 21);"> </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">192.168.1.0/24</font>`<font style="color:rgb(15, 17, 21);">，两种模式都可以。但为了演示全局地址池（并方便后续添加静态绑定），你使用了全局模式。</font>

**<font style="color:rgb(15, 17, 21);">关键配置（穿插）</font>**<font style="color:rgb(15, 17, 21);">：</font>

<font style="color:rgb(15, 17, 21);">bash</font>

```plain
[R1] dhcp enable
[R1] ip pool lan
[R1-ip-pool-lan] network 192.168.1.0 mask 24
[R1-ip-pool-lan] gateway-list 192.168.1.1
[R1-ip-pool-lan] dns-list 192.168.1.1      # 注意：这里指向 R1 自身，用于 DNS 代理
[R1-ip-pool-lan] lease day 1
[R1-ip-pool-lan] quit
[R1] interface GigabitEthernet0/0/0
[R1-GigabitEthernet0/0/0] ip address 192.168.1.1 24
[R1-GigabitEthernet0/0/0] dhcp select global
```

**<font style="color:rgb(15, 17, 21);">抓包细节</font>**<font style="color:rgb(15, 17, 21);">：在 PC 侧抓</font><font style="color:rgb(15, 17, 21);"> </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">bootp</font>`<font style="color:rgb(15, 17, 21);"> </font><font style="color:rgb(15, 17, 21);">或</font><font style="color:rgb(15, 17, 21);"> </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">dhcp</font>`<font style="color:rgb(15, 17, 21);">，你会看到 DORA 四个报文。注意 DHCP Offer 中</font><font style="color:rgb(15, 17, 21);"> </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">Option 6 (DNS)</font>`<font style="color:rgb(15, 17, 21);"> </font><font style="color:rgb(15, 17, 21);">的值应该是</font><font style="color:rgb(15, 17, 21);"> </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">192.168.1.1</font>`<font style="color:rgb(15, 17, 21);">。</font>

**<font style="color:rgb(15, 17, 21);">验证命令</font>**<font style="color:rgb(15, 17, 21);">：</font>

<font style="color:rgb(15, 17, 21);">bash</font>

```plain
display dhcp server lease        # 查看已分配的 IP
display ip pool name lan used    # 查看地址池使用情况
```

**<font style="color:rgb(15, 17, 21);">为什么这里必须把 DNS 指向 R1？</font>**<font style="color:rgb(15, 17, 21);">  
</font><font style="color:rgb(15, 17, 21);">因为后续我们要用 R1 做 DNS 代理，如果 PC 直接把 DNS 请求发往外网（如</font><font style="color:rgb(15, 17, 21);"> </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">2.0.0.66</font>`<font style="color:rgb(15, 17, 21);">），R1 的代理就不会生效。</font>

---

## <font style="color:rgb(15, 17, 21);">二、NAT：内网访问外网 + 外网访问内网服务器</font>
### <font style="color:rgb(15, 17, 21);">1. Easy IP（内网上网）</font>
+ **<font style="color:rgb(15, 17, 21);">原理</font>**<font style="color:rgb(15, 17, 21);">：将所有内网流量的源 IP 转换为出接口 IP（</font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">12.0.0.1</font>`<font style="color:rgb(15, 17, 21);">），端口随机。</font>

**<font style="color:rgb(15, 17, 21);">配置</font>**<font style="color:rgb(15, 17, 21);">：</font>

+ <font style="color:rgb(15, 17, 21);">bash</font>

```plain
[R1] acl 2000
[R1-acl-basic-2000] rule 5 permit source 192.168.1.0 0.0.0.255
[R1-acl-basic-2000] quit
[R1] interface GigabitEthernet0/0/1
[R1-GigabitEthernet0/0/1] ip address 12.0.0.1 24
[R1-GigabitEthernet0/0/1] nat outbound 2000
```

**<font style="color:rgb(15, 17, 21);">抓包验证</font>**<font style="color:rgb(15, 17, 21);">：在 R1 的外网接口抓包，内网 PC ping</font><font style="color:rgb(15, 17, 21);"> </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">2.0.0.50</font>`<font style="color:rgb(15, 17, 21);">，抓到的 ICMP 请求源 IP 应该是</font><font style="color:rgb(15, 17, 21);"> </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">12.0.0.1</font>`<font style="color:rgb(15, 17, 21);">。</font>

### <font style="color:rgb(15, 17, 21);">2. NAT Server（端口映射）</font>
+ **<font style="color:rgb(15, 17, 21);">需求</font>**<font style="color:rgb(15, 17, 21);">：外网用户通过</font><font style="color:rgb(15, 17, 21);"> </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">http://12.0.0.1</font>`<font style="color:rgb(15, 17, 21);"> </font><font style="color:rgb(15, 17, 21);">访问内网 Web 服务器</font><font style="color:rgb(15, 17, 21);"> </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">192.168.1.100</font>`<font style="color:rgb(15, 17, 21);">。</font>

**<font style="color:rgb(15, 17, 21);">配置</font>**<font style="color:rgb(15, 17, 21);">：</font>

+ <font style="color:rgb(15, 17, 21);">bash</font>

```plain
[R1] interface GigabitEthernet0/0/1
[R1-GigabitEthernet0/0/1] nat server protocol tcp global 12.0.0.1 80 inside 192.168.1.100 80
[R1-GigabitEthernet0/0/1] nat server protocol tcp global 12.0.0.1 443 inside 192.168.1.100 443
```

**<font style="color:rgb(15, 17, 21);">抓包验证</font>**<font style="color:rgb(15, 17, 21);">：外网主机访问</font><font style="color:rgb(15, 17, 21);"> </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">http://12.0.0.1</font>`<font style="color:rgb(15, 17, 21);">，在 R1 外网接口抓包，目标 IP 为</font><font style="color:rgb(15, 17, 21);"> </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">12.0.0.1</font>`<font style="color:rgb(15, 17, 21);"> </font><font style="color:rgb(15, 17, 21);">端口 80，经过 NAT 后内网接口的目标 IP 变为</font><font style="color:rgb(15, 17, 21);"> </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">192.168.1.100</font>`<font style="color:rgb(15, 17, 21);">。</font>

### <font style="color:rgb(15, 17, 21);">3. 额外的映射：</font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">12.0.0.66</font>`<font style="color:rgb(15, 17, 21);"> </font><font style="color:rgb(15, 17, 21);">作为 secondary IP</font>
+ **<font style="color:rgb(15, 17, 21);">背景</font>**<font style="color:rgb(15, 17, 21);">：你需要将 ICMP 和 Web 服务也映射到同一台内网服务器，但公网 IP 是</font><font style="color:rgb(15, 17, 21);"> </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">12.0.0.66</font>`<font style="color:rgb(15, 17, 21);">（不是接口主 IP）。</font>

**<font style="color:rgb(15, 17, 21);">解决</font>**<font style="color:rgb(15, 17, 21);">：添加 secondary IP，并配置 NAT Server。</font>

+ <font style="color:rgb(15, 17, 21);">bash</font>

```plain
[R1] interface GigabitEthernet0/0/1
[R1-GigabitEthernet0/0/1] ip address 12.0.0.66 24 sub    # 添加从地址
[R1-GigabitEthernet0/0/1] nat server protocol icmp global 12.0.0.66 inside 192.168.1.100
[R1-GigabitEthernet0/0/1] nat server protocol tcp global 12.0.0.66 www inside 192.168.1.100 www
```

**<font style="color:rgb(15, 17, 21);">注意</font>**<font style="color:rgb(15, 17, 21);">：secondary IP 必须配置，否则外网 ARP 无法解析</font><font style="color:rgb(15, 17, 21);"> </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">12.0.0.66</font>`<font style="color:rgb(15, 17, 21);">。</font>

**<font style="color:rgb(15, 17, 21);">验证</font>**<font style="color:rgb(15, 17, 21);">：外网主机</font><font style="color:rgb(15, 17, 21);"> </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">ping 12.0.0.66</font>`<font style="color:rgb(15, 17, 21);"> </font><font style="color:rgb(15, 17, 21);">应通（实际由内网</font><font style="color:rgb(15, 17, 21);"> </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">192.168.1.100</font>`<font style="color:rgb(15, 17, 21);"> </font><font style="color:rgb(15, 17, 21);">回应）。</font>

### <font style="color:rgb(15, 17, 21);">4. 缺省路由（指向 ISP）</font>
<font style="color:rgb(15, 17, 21);">bash</font>

<font style="color:rgb(15, 17, 21);">[R1] ip route-static 0.0.0.0 0 12.0.0.2</font>

**<font style="color:rgb(15, 17, 21);">ISP 配置</font>**<font style="color:rgb(15, 17, 21);">（无静态路由，依靠直连）：</font>

<font style="color:rgb(15, 17, 21);">bash</font>

```plain
[ISP] interface GigabitEthernet0/0/0
[ISP-GigabitEthernet0/0/0] ip address 12.0.0.2 24
[ISP] interface GigabitEthernet0/0/1
[ISP-GigabitEthernet0/0/1] ip address 2.0.0.1 24
```

<font style="color:rgb(15, 17, 21);">ISP 拥有直连路由</font><font style="color:rgb(15, 17, 21);"> </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">2.0.0.0/24</font>`<font style="color:rgb(15, 17, 21);"> </font><font style="color:rgb(15, 17, 21);">和</font><font style="color:rgb(15, 17, 21);"> </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">12.0.0.0/24</font>`<font style="color:rgb(15, 17, 21);">，因此可以转发 R1 发往</font><font style="color:rgb(15, 17, 21);"> </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">2.0.0.x</font>`<font style="color:rgb(15, 17, 21);"> </font><font style="color:rgb(15, 17, 21);">的流量。</font>

---

## <font style="color:rgb(15, 17, 21);">三、ACL：精细化访问控制</font>
### <font style="color:rgb(15, 17, 21);">1. 高级 ACL 语法</font>
**<font style="color:rgb(15, 17, 21);">匹配 ICMP 回显请求（ping）</font>**<font style="color:rgb(15, 17, 21);">：</font>

<font style="color:rgb(15, 17, 21);">bash</font>

```plain
rule 5 deny icmp source 192.168.1.10 0 destination 2.0.0.66 0 icmp-type echo
```

    - `<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">icmp-type echo</font>`<font style="color:rgb(15, 17, 21);"> </font><font style="color:rgb(15, 17, 21);">表示 ping 请求。</font>
    - <font style="color:rgb(15, 17, 21);">目的 IP</font><font style="color:rgb(15, 17, 21);"> </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">2.0.0.66</font>`<font style="color:rgb(15, 17, 21);"> </font><font style="color:rgb(15, 17, 21);">是外网 DNS 服务器。</font>

**<font style="color:rgb(15, 17, 21);">匹配 TCP 端口（FTP）</font>**<font style="color:rgb(15, 17, 21);">：</font>

<font style="color:rgb(15, 17, 21);">bash</font>

```plain
rule 5 deny tcp source 192.168.1.30 0 destination 2.0.0.20 0 destination-port eq 21
```

+ <font style="color:rgb(15, 17, 21);">这条规则是</font><font style="color:rgb(15, 17, 21);"> </font>**<font style="color:rgb(15, 17, 21);">高级 ACL（编号 3000-3999）</font>**<font style="color:rgb(15, 17, 21);"> </font><font style="color:rgb(15, 17, 21);">中的一条，用于</font>**<font style="color:rgb(15, 17, 21);">禁止源 IP 为</font>****<font style="color:rgb(15, 17, 21);"> </font>**`**<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">192.168.1.30</font>**`**<font style="color:rgb(15, 17, 21);"> </font>****<font style="color:rgb(15, 17, 21);">的主机向目的 IP</font>****<font style="color:rgb(15, 17, 21);"> </font>**`**<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">2.0.0.20</font>**`**<font style="color:rgb(15, 17, 21);"> </font>****<font style="color:rgb(15, 17, 21);">的 TCP 端口 21（FTP 控制端口）发起连接</font>**<font style="color:rgb(15, 17, 21);">。它常用于限制特定客户端访问 FTP 服务。</font>

### <font style="color:rgb(15, 17, 21);">字段逐项拆解</font>
| <font style="color:rgb(15, 17, 21);">字段</font> | <font style="color:rgb(15, 17, 21);">值</font> | <font style="color:rgb(15, 17, 21);">含义</font> |
| --- | --- | --- |
| `<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">rule 5</font>` | <font style="color:rgb(15, 17, 21);">规则序号</font> | <font style="color:rgb(15, 17, 21);">数值越小，匹配优先级越高（先于</font><font style="color:rgb(15, 17, 21);"> </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">rule 100</font>`<br/><font style="color:rgb(15, 17, 21);"> </font><font style="color:rgb(15, 17, 21);">等）</font> |
| `<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">deny</font>` | <font style="color:rgb(15, 17, 21);">动作</font> | <font style="color:rgb(15, 17, 21);">拒绝匹配的报文</font> |
| `<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">tcp</font>` | <font style="color:rgb(15, 17, 21);">协议类型</font> | <font style="color:rgb(15, 17, 21);">匹配 TCP 报文</font> |
| `<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">source 192.168.1.30 0</font>` | <font style="color:rgb(15, 17, 21);">源地址</font> | <font style="color:rgb(15, 17, 21);">IP =</font><font style="color:rgb(15, 17, 21);"> </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">192.168.1.30</font>`<br/><font style="color:rgb(15, 17, 21);">，通配符掩码</font><font style="color:rgb(15, 17, 21);"> </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">0</font>`<br/><font style="color:rgb(15, 17, 21);"> </font><font style="color:rgb(15, 17, 21);">表示精确匹配该主机</font> |
| `<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">destination 2.0.0.20 0</font>` | <font style="color:rgb(15, 17, 21);">目的地址</font> | <font style="color:rgb(15, 17, 21);">IP =</font><font style="color:rgb(15, 17, 21);"> </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">2.0.0.20</font>`<br/><font style="color:rgb(15, 17, 21);">，精确匹配</font> |
| `<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">destination-port eq 21</font>` | <font style="color:rgb(15, 17, 21);">目的端口</font> | <font style="color:rgb(15, 17, 21);">等于 21（FTP 控制端口）</font> |


**<font style="color:rgb(15, 17, 21);">通配符掩码说明</font>**<font style="color:rgb(15, 17, 21);">：</font>

+ `<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">0</font>`<font style="color:rgb(15, 17, 21);"> </font><font style="color:rgb(15, 17, 21);">表示“必须匹配”，</font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">1</font>`<font style="color:rgb(15, 17, 21);"> </font><font style="color:rgb(15, 17, 21);">表示“忽略”。</font>
+ `<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">192.168.1.30 0</font>`<font style="color:rgb(15, 17, 21);"> </font><font style="color:rgb(15, 17, 21);">等价于</font><font style="color:rgb(15, 17, 21);"> </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">192.168.1.30 255.255.255.255</font>`<font style="color:rgb(15, 17, 21);">（正掩码），即精确匹配该 IP。</font>
+ <font style="color:rgb(15, 17, 21);">如果是</font><font style="color:rgb(15, 17, 21);"> </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">192.168.1.0 0.0.0.255</font>`<font style="color:rgb(15, 17, 21);">，则匹配整个</font><font style="color:rgb(15, 17, 21);"> </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">192.168.1.0/24</font>`<font style="color:rgb(15, 17, 21);"> </font><font style="color:rgb(15, 17, 21);">网段。</font>

### <font style="color:rgb(15, 17, 21);">实际效果</font>
+ <font style="color:rgb(15, 17, 21);">当 Client1（假设 IP =</font><font style="color:rgb(15, 17, 21);"> </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">192.168.1.30</font>`<font style="color:rgb(15, 17, 21);">）尝试通过 FTP 客户端连接</font><font style="color:rgb(15, 17, 21);"> </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">2.0.0.20</font>`<font style="color:rgb(15, 17, 21);"> </font><font style="color:rgb(15, 17, 21);">的 21 端口时，</font>**<font style="color:rgb(15, 17, 21);">TCP SYN 报文会被 R1 丢弃</font>**<font style="color:rgb(15, 17, 21);">，连接无法建立。</font>
+ <font style="color:rgb(15, 17, 21);">FTP 数据端口（20）或其他端口不受影响（因为只匹配</font><font style="color:rgb(15, 17, 21);"> </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">eq 21</font>`<font style="color:rgb(15, 17, 21);">）。</font>

### <font style="color:rgb(15, 17, 21);">抓包验证</font>
+ <font style="color:rgb(15, 17, 21);">在 R1 的内网接口（</font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">GE0/0/0</font>`<font style="color:rgb(15, 17, 21);">）入方向抓包，过滤</font><font style="color:rgb(15, 17, 21);"> </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">tcp.port==21 and ip.src==192.168.1.30 and ip.dst==2.0.0.20</font>`<font style="color:rgb(15, 17, 21);">。</font>
+ <font style="color:rgb(15, 17, 21);">你会看到 Client1 发出的 SYN 报文，但没有收到 SYN-ACK（被 ACL 丢弃）。</font>

### <font style="color:rgb(15, 17, 21);">2. 必须添加放行规则</font>
<font style="color:rgb(15, 17, 21);">每个 ACL 末尾默认有</font><font style="color:rgb(15, 17, 21);"> </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">deny any any</font>`<font style="color:rgb(15, 17, 21);">，因此必须加：</font>

<font style="color:rgb(15, 17, 21);">bash</font>

<font style="color:rgb(15, 17, 21);">rule 100 permit ip</font>

### <font style="color:rgb(15, 17, 21);">3. 应用到接口</font>
<font style="color:rgb(15, 17, 21);">bash</font>

```plain
[R1] interface GigabitEthernet0/0/0
[R1-GigabitEthernet0/0/0] traffic-filter inbound acl 3000
[R1-GigabitEthernet0/0/0] traffic-filter inbound acl 3001
[R1-GigabitEthernet0/0/0] traffic-filter inbound acl 3002
```

**<font style="color:rgb(15, 17, 21);">抓包验证</font>**<font style="color:rgb(15, 17, 21);">：在 R1 内网接口抓包，PC1 ping</font><font style="color:rgb(15, 17, 21);"> </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">2.0.0.66</font>`<font style="color:rgb(15, 17, 21);">，你会看到 ICMP Echo Request 进入接口，但没有 Echo Reply 返回（被 ACL 丢弃）。</font>

**<font style="color:rgb(15, 17, 21);">注意</font>**<font style="color:rgb(15, 17, 21);">：ACL 中的源 IP（如</font><font style="color:rgb(15, 17, 21);"> </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">192.168.1.10</font>`<font style="color:rgb(15, 17, 21);">）必须与 PC 实际获得的 IP 一致。如果 DHCP 是动态分配，这些 IP 可能不对应 PC1/PC2/Client1。</font>**<font style="color:rgb(15, 17, 21);">解决方法</font>**<font style="color:rgb(15, 17, 21);">：</font>

+ <font style="color:rgb(15, 17, 21);">方案一：在</font><font style="color:rgb(15, 17, 21);"> </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">ip pool lan</font>`<font style="color:rgb(15, 17, 21);"> </font><font style="color:rgb(15, 17, 21);">中使用</font><font style="color:rgb(15, 17, 21);"> </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">static-bind</font>`<font style="color:rgb(15, 17, 21);"> </font><font style="color:rgb(15, 17, 21);">绑定 MAC 地址。</font>
+ <font style="color:rgb(15, 17, 21);">方案二：将 ACL 中的源 IP 改为匹配整个网段（但那样会限制所有主机）。</font>

---

## <font style="color:rgb(15, 17, 21);">四、DNS 代理：让内网主机通过 R1 解析域名</font>
### <font style="color:rgb(15, 17, 21);">1. 为什么要用 DNS 代理？</font>
+ <font style="color:rgb(15, 17, 21);">如果 PC 直接向外网 DNS（</font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">2.0.0.66</font>`<font style="color:rgb(15, 17, 21);">）发请求，R1 无法控制或缓存域名解析。</font>
+ <font style="color:rgb(15, 17, 21);">通过代理，R1 可以缓存解析结果、实现域名过滤、避免外网 DNS 变更影响。</font>

### <font style="color:rgb(15, 17, 21);">2. 配置 DNS 代理</font>
<font style="color:rgb(15, 17, 21);">bash</font>

```plain
[R1] dns resolve               # 开启路由器的 DNS 解析功能（作为客户端）
[R1] dns server 2.0.0.66      # 指定上游 DNS 服务器的 IP
[R1] dns proxy enable          # 开启 DNS 代理功能
```

### <font style="color:rgb(15, 17, 21);">常见错误</font>
+ **<font style="color:rgb(15, 17, 21);">内网 PC 的 DNS 指向外网 DNS</font>**<font style="color:rgb(15, 17, 21);">：代理不会生效，因为 PC 直接向外网发起请求。</font>
+ **<font style="color:rgb(15, 17, 21);">未配置</font>****<font style="color:rgb(15, 17, 21);"> </font>**`**<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">dns resolve</font>**`<font style="color:rgb(15, 17, 21);">：路由器无法作为 DNS 客户端向上游查询。</font>
+ **<font style="color:rgb(15, 17, 21);">未配置 </font>**`**<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">dns server</font>**`<font style="color:rgb(15, 17, 21);">：代理不知道转发给谁。</font>

### <font style="color:rgb(15, 17, 21);">3. 确保 PC 的 DNS 指向 R1</font>
+ <font style="color:rgb(15, 17, 21);">在 DHCP 地址池中已经配置了</font><font style="color:rgb(15, 17, 21);"> </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">dns-list 192.168.1.1</font>`<font style="color:rgb(15, 17, 21);">。</font>

<font style="color:rgb(15, 17, 21);">如果之前配成了</font><font style="color:rgb(15, 17, 21);"> </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">2.0.0.66</font>`<font style="color:rgb(15, 17, 21);">，请修改：</font>

+ <font style="color:rgb(15, 17, 21);">bash</font>

```plain
[R1] ip pool lan
[R1-ip-pool-lan] dns-list 192.168.1.1
```

### <font style="color:rgb(15, 17, 21);">4. 抓包验证 DNS 代理</font>
+ **<font style="color:rgb(15, 17, 21);">内网接口抓包</font>**<font style="color:rgb(15, 17, 21);">：过滤</font><font style="color:rgb(15, 17, 21);"> </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">udp.port==53</font>`<font style="color:rgb(15, 17, 21);">，PC 发起的 DNS 请求目的 IP 应为</font><font style="color:rgb(15, 17, 21);"> </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">192.168.1.1</font>`<font style="color:rgb(15, 17, 21);">。</font>
+ **<font style="color:rgb(15, 17, 21);">外网接口抓包</font>**<font style="color:rgb(15, 17, 21);">：R1 转发给</font><font style="color:rgb(15, 17, 21);"> </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">2.0.0.66</font>`<font style="color:rgb(15, 17, 21);"> </font><font style="color:rgb(15, 17, 21);">的请求，源 IP 为</font><font style="color:rgb(15, 17, 21);"> </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">12.0.0.1</font>`<font style="color:rgb(15, 17, 21);">，目的 IP</font><font style="color:rgb(15, 17, 21);"> </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">2.0.0.66</font>`<font style="color:rgb(15, 17, 21);">。</font>
+ **<font style="color:rgb(15, 17, 21);">响应</font>**<font style="color:rgb(15, 17, 21);">：外网 DNS 返回结果，R1 再转发给 PC。</font>

**<font style="color:rgb(15, 17, 21);">验证命令</font>**<font style="color:rgb(15, 17, 21);">：</font>

<font style="color:rgb(15, 17, 21);">bash</font>

<font style="color:rgb(15, 17, 21);">display dns proxy dynamic        # 查看代理缓存</font>

---

## <font style="color:rgb(15, 17, 21);">五、整合与验证清单</font>
<font style="color:rgb(15, 17, 21);">按以上知识点配置后，进行以下测试：</font>

| <font style="color:rgb(15, 17, 21);">测试项</font> | <font style="color:rgb(15, 17, 21);">预期结果</font> | <font style="color:rgb(15, 17, 21);">验证命令/动作</font> |
| --- | --- | --- |
| <font style="color:rgb(15, 17, 21);">PC1 获取 IP</font> | <font style="color:rgb(15, 17, 21);">IP 在 192.168.1.x，网关 192.168.1.1，DNS 192.168.1.1</font> | `<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">ipconfig</font>` |
| <font style="color:rgb(15, 17, 21);">PC1 ping 2.0.0.50（淘宝）</font> | <font style="color:rgb(15, 17, 21);">通</font> | `<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">ping 2.0.0.50</font>` |
| <font style="color:rgb(15, 17, 21);">PC1 ping 2.0.0.66（DNS）</font> | <font style="color:rgb(15, 17, 21);">不通（ACL 3000）</font> | `<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">ping 2.0.0.66</font>` |
| <font style="color:rgb(15, 17, 21);">PC2 ping 2.0.0.50</font> | <font style="color:rgb(15, 17, 21);">不通（ACL 3001）</font> | `<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">ping 2.0.0.50</font>` |
| <font style="color:rgb(15, 17, 21);">Client1 访问 ftp://2.0.0.20</font> | <font style="color:rgb(15, 17, 21);">失败（ACL 3002）</font> | <font style="color:rgb(15, 17, 21);">使用 FTP 客户端</font> |
| <font style="color:rgb(15, 17, 21);">外网主机访问</font><font style="color:rgb(15, 17, 21);"> </font>[<font style="color:rgb(57, 100, 254);">http://12.0.0.1</font>](http://12.0.0.1/) | <font style="color:rgb(15, 17, 21);">显示内网 Web 页面</font> | <font style="color:rgb(15, 17, 21);">浏览器</font> |
| <font style="color:rgb(15, 17, 21);">外网主机 ping 12.0.0.66</font> | <font style="color:rgb(15, 17, 21);">通（映射到内网 192.168.1.100）</font> | `<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">ping 12.0.0.66</font>` |
| <font style="color:rgb(15, 17, 21);">内网 PC nslookup</font><font style="color:rgb(15, 17, 21);"> </font>[<font style="color:rgb(57, 100, 254);">www.taobao.com</font>](https://www.taobao.com/) | <font style="color:rgb(15, 17, 21);">返回 2.0.0.50</font> | `<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">nslookup www.taobao.com</font>` |


---

## <font style="color:rgb(15, 17, 21);">六、拓展思考（不止于本实验）</font>
+ **<font style="color:rgb(15, 17, 21);">如果 ISP 不提供 DNS 服务</font>**<font style="color:rgb(15, 17, 21);">：可以在 R1 上配置静态域名解析（</font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">ip host</font>`<font style="color:rgb(15, 17, 21);">）或使用公共 DNS（如</font><font style="color:rgb(15, 17, 21);"> </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">8.8.8.8</font>`<font style="color:rgb(15, 17, 21);">）。</font>
+ **<font style="color:rgb(15, 17, 21);">如何防止内网用户私自更改 DNS</font>**<font style="color:rgb(15, 17, 21);">：在 R1 上配置策略路由或 ACL 强制重定向 DNS 流量。</font>
+ **<font style="color:rgb(15, 17, 21);">NAT 与 ACL 的顺序</font>**<font style="color:rgb(15, 17, 21);">：入方向 ACL 先于 NAT（先过滤后转换），出方向 NAT 先于 ACL（先转换后过滤）。</font>
+ **<font style="color:rgb(15, 17, 21);">secondary IP 的替代方案</font>**<font style="color:rgb(15, 17, 21);">：可以使用 </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">nat server protocol tcp global 12.0.0.1 8080 inside 192.168.1.100 80</font>`<font style="color:rgb(15, 17, 21);">，用一个端口映射多个服务。</font>
