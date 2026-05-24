<img src="https://cdn.nlark.com/yuque/0/2026/png/50508853/1776085925296-54c359e5-1f77-4479-81c8-4af795240751.png" width="947.3333333333334" title="" crop="0,0,1,1" id="u386fc9c5" class="ne-image">

<img src="https://cdn.nlark.com/yuque/0/2026/png/50508853/1776419839308-5b9f0a98-c788-4f81-96fa-8f497e6c8d23.png" width="1480.6666666666667" title="" crop="0,0,1,1" id="ua3495eac" class="ne-image">

## <font style="color:rgb(15, 17, 21);">一、实验结论</font>
<font style="color:rgb(15, 17, 21);">本次实验成功搭建了一个</font>**<font style="color:rgb(15, 17, 21);">企业园区网 + 无线 + PPPoE拨号 + NAT上网 + ISP多区域OSPF</font>**<font style="color:rgb(15, 17, 21);">的完整网络环境。核心结论如下：</font>

| <font style="color:rgb(15, 17, 21);">模块</font> | <font style="color:rgb(15, 17, 21);">结论</font> |
| --- | --- |
| <font style="color:rgb(15, 17, 21);">内网二层</font> | <font style="color:rgb(15, 17, 21);">VLAN划分正确，Trunk放行所有业务VLAN，STP根桥为SW3。</font> |
| <font style="color:rgb(15, 17, 21);">内网三层</font> | <font style="color:rgb(15, 17, 21);">SW3实现VLAN间路由，OSPF 1互通，缺省路由从R1注入。</font> |
| <font style="color:rgb(15, 17, 21);">无线网络</font> | <font style="color:rgb(15, 17, 21);">AP通过DHCP Option 43发现AC1，直接转发模式，业务VLAN 50/60。</font> |
| <font style="color:rgb(15, 17, 21);">出口网络</font> | <font style="color:rgb(15, 17, 21);">R1通过PPPoE拨号获取公网IP，PAT实现内网上网，ACL拒绝VLAN20的HTTP。</font> |
| <font style="color:rgb(15, 17, 21);">ISP网络</font> | <font style="color:rgb(15, 17, 21);">R2/R3/R4运行OSPF 2，DNS和HTTP/FTP服务器可访问。</font> |


---

## <font style="color:rgb(15, 17, 21);">二、架构总览</font>
<font style="color:rgb(15, 17, 21);">text</font>

```plain
内网（OSPF 1）                    外网（OSPF 2）
PC1(10) ─ SW1 ─┐
PC2(20) ─ SW1 ─┼─ SW3 ─ SW5 ─ R1 ═══ R2 ─ R3 ─ DNS(3.0.0.100)
PC3(30) ─ SW2 ─┘         │        PPPoE     └─ R4 ─ Server(4.0.0.100)
PC4(40) ─ SW2 ───────────┘
AP1/2 ─ SW1/SW2 ─ SW3 ─ AC1
```

---

## <font style="color:rgb(15, 17, 21);">三、关键配置与原理拆解</font>
### <font style="color:rgb(15, 17, 21);">1. Eth-Trunk（SW3 </font><font style="color:rgb(15, 17, 21);">↔</font><font style="color:rgb(15, 17, 21);"> SW5）</font>
<font style="color:rgb(15, 17, 21);">bash</font>

```plain
interface Eth-Trunk1
 mode lacp-static                 # 静态LACP，允许备份链路
 max active-linknumber 2          # 最多2条活动链路
 trunkport GigabitEthernet 0/0/21 to 0/0/23
 port link-type trunk
 port trunk allow-pass vlan 35    # 只放行VLAN 35，承载三层互联
 port trunk pvid vlan 35          # 无标签帧打上VLAN 35
```

**<font style="color:rgb(15, 17, 21);">原理</font>**<font style="color:rgb(15, 17, 21);">：Eth-Trunk1被当作</font>**<font style="color:rgb(15, 17, 21);">三层路由接口</font>**<font style="color:rgb(15, 17, 21);">使用（两端配置Vlanif35 IP），VLAN 35仅作为二层承载。其他VLAN的流量经过路由后，从Vlanif35发出时打上VLAN 35标签，因此不需要放行其他VLAN。</font>

**<font style="color:rgb(15, 17, 21);">验证</font>**<font style="color:rgb(15, 17, 21);">：</font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">display eth-trunk 1</font>`<font style="color:rgb(15, 17, 21);"> </font><font style="color:rgb(15, 17, 21);">→ 成员端口状态为Selected。</font>

---

### <font style="color:rgb(15, 17, 21);">2. Trunk放行规则（修正后）</font>
| <font style="color:rgb(15, 17, 21);">链路</font> | <font style="color:rgb(15, 17, 21);">放行VLAN</font> | <font style="color:rgb(15, 17, 21);">原因</font> |
| --- | --- | --- |
| <font style="color:rgb(15, 17, 21);">SW1-SW3 / SW2-SW3</font> | <font style="color:rgb(15, 17, 21);">10,20,30,40,50,60,100</font> | <font style="color:rgb(15, 17, 21);">所有业务VLAN + 管理VLAN</font> |
| <font style="color:rgb(15, 17, 21);">SW1-SW2</font> | <font style="color:rgb(15, 17, 21);">10,20,30,40,50,60,100</font> | <font style="color:rgb(15, 17, 21);">实现冗余备份（当SW1-SW3故障时）</font> |
| <font style="color:rgb(15, 17, 21);">SW1-AP1</font> | <font style="color:rgb(15, 17, 21);">Trunk PVID 100，放行50,100</font> | <font style="color:rgb(15, 17, 21);">直接转发模式：管理VLAN 100 + 业务VLAN 50</font> |
| <font style="color:rgb(15, 17, 21);">SW2-AP2</font> | <font style="color:rgb(15, 17, 21);">Trunk PVID 100，放行60,100</font> | <font style="color:rgb(15, 17, 21);">直接转发模式：管理VLAN 100 + 业务VLAN 60</font> |


**<font style="color:rgb(15, 17, 21);">验证</font>**<font style="color:rgb(15, 17, 21);">：</font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">display interface trunk</font>`<font style="color:rgb(15, 17, 21);"> </font><font style="color:rgb(15, 17, 21);">→ 查看Allowed VLAN列表。</font>

---

### <font style="color:rgb(15, 17, 21);">3. VLAN间路由与DHCP</font>
<font style="color:rgb(15, 17, 21);">bash</font>

```plain
interface Vlanif10
 ip address 192.168.10.1 255.255.255.0
 dhcp select interface          # 接口地址池模式
 dhcp server dns-list 3.0.0.100 # 下放DNS
```

**<font style="color:rgb(15, 17, 21);">原理</font>**<font style="color:rgb(15, 17, 21);">：DHCP接口地址池自动从接口IP所在网段分配地址，无需手动配置地址池范围。</font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">dns-list</font>`<font style="color:rgb(15, 17, 21);">必须配置，否则PC无法解析域名。</font>

**<font style="color:rgb(15, 17, 21);">验证</font>**<font style="color:rgb(15, 17, 21);">：PC上</font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">ipconfig</font>`<font style="color:rgb(15, 17, 21);">应显示DNS为3.0.0.100。</font>

---

### <font style="color:rgb(15, 17, 21);">4. 无线直接转发与Option 43</font>
<font style="color:rgb(15, 17, 21);">bash</font>

```plain
# SW3 Vlanif100
dhcp server option 43 sub-option 2 ip-address 192.168.51.1

# AC1
capwap source interface vlanif 51
vap-profile name Office
 forward-mode direct-forward     # 直接转发
 service-vlan vlan-id 50
```

**<font style="color:rgb(15, 17, 21);">原理</font>**<font style="color:rgb(15, 17, 21);">：</font>

+ <font style="color:rgb(15, 17, 21);">Option 43告诉AP：AC的IP是192.168.51.1。</font>
+ <font style="color:rgb(15, 17, 21);">直接转发模式下，用户数据由AP直接打上业务VLAN标签发送，不经过AC。</font>

**<font style="color:rgb(15, 17, 21);">常见错误</font>**<font style="color:rgb(15, 17, 21);">：AP接口误配为Access模式，导致业务VLAN无法透传。正确应为Trunk，PVID为管理VLAN，放行业务VLAN。</font>

**<font style="color:rgb(15, 17, 21);">验证</font>**<font style="color:rgb(15, 17, 21);">：</font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">display ap all</font>`<font style="color:rgb(15, 17, 21);"> </font><font style="color:rgb(15, 17, 21);">→ AP状态为normal；STA获取IP应为业务VLAN网段。</font>

---

### <font style="color:rgb(15, 17, 21);">5. PPPoE拨号与NAT</font>
<font style="color:rgb(15, 17, 21);">bash</font>

```plain
# R1客户端
interface Dialer1
 ppp chap user wakin
 ppp chap password cipher 666.com
 ip address ppp-negotiate
 nat outbound 3000
 dialer-group 1
dialer-rule 1 ip permit

# R2服务端
ip pool pppoe
 gateway 66.66.66.1
 section 0 66.66.66.10 66.66.66.200
interface Virtual-Template1
 ppp authentication-mode chap
 peer default ip address pool pppoe
```

**<font style="color:rgb(15, 17, 21);">关键点</font>**<font style="color:rgb(15, 17, 21);">：</font>

+ <font style="color:rgb(15, 17, 21);">客户端</font>**<font style="color:rgb(15, 17, 21);">不能</font>**<font style="color:rgb(15, 17, 21);">配置</font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">ppp authentication-mode chap</font>`<font style="color:rgb(15, 17, 21);">（那是服务端用的）。</font>
+ <font style="color:rgb(15, 17, 21);">必须配置</font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">dialer-rule</font>`<font style="color:rgb(15, 17, 21);">和</font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">dialer-group</font>`<font style="color:rgb(15, 17, 21);">才能触发拨号。</font>
+ <font style="color:rgb(15, 17, 21);">服务端必须配置</font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">peer default ip address pool</font>`<font style="color:rgb(15, 17, 21);">才能分配IP。</font>

**<font style="color:rgb(15, 17, 21);">验证</font>**<font style="color:rgb(15, 17, 21);">：</font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">display interface Dialer1</font>`<font style="color:rgb(15, 17, 21);"> </font><font style="color:rgb(15, 17, 21);">→ 应有</font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">Internet Address: 66.66.66.254/32</font>`<font style="color:rgb(15, 17, 21);">。</font>

---

### <font style="color:rgb(15, 17, 21);">6. ACL（拒绝VLAN20的HTTP）</font>
<font style="color:rgb(15, 17, 21);">bash</font>

```plain
acl number 3000
 rule 5 deny tcp source 192.168.20.0 0.0.0.255 destination-port eq 80
 rule 10 permit ip
```

**<font style="color:rgb(15, 17, 21);">注意</font>**<font style="color:rgb(15, 17, 21);">：需求是“拒绝VLAN20的主机通过浏览器访问HTTP服务器”，所以只拒绝TCP 80端口，不影响其他流量（如ping、FTP）。</font>

**<font style="color:rgb(15, 17, 21);">验证</font>**<font style="color:rgb(15, 17, 21);">：PC2（VLAN20）</font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">ping 4.0.0.100</font>`<font style="color:rgb(15, 17, 21);">通，但浏览器访问</font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">http://4.0.0.100</font>`<font style="color:rgb(15, 17, 21);">被拒绝。</font>

---

### <font style="color:rgb(15, 17, 21);">7. 缺省路由与OSPF注入</font>
<font style="color:rgb(15, 17, 21);">bash</font>

```plain
# R1
ip route-static 0.0.0.0 0.0.0.0 Dialer1
ospf 1
 default-route-advertise always

# SW3上应学到默认路由
display ip routing-table 0.0.0.0
```

**<font style="color:rgb(15, 17, 21);">原理</font>**<font style="color:rgb(15, 17, 21);">：</font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">default-route-advertise always</font>`<font style="color:rgb(15, 17, 21);">强制R1在OSPF 1中发布缺省路由，内网设备即可通过OSPF学习到外网出口。</font>

**<font style="color:rgb(15, 17, 21);">排错</font>**<font style="color:rgb(15, 17, 21);">：如果SW3没有默认路由，检查OSPF邻居状态（</font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">display ospf peer brief</font>`<font style="color:rgb(15, 17, 21);">）是否为Full。若邻居正常但仍无，可临时添加静态默认路由。</font>

---

## <font style="color:rgb(15, 17, 21);">四、错误总结与教训</font>
| <font style="color:rgb(15, 17, 21);">错误</font> | <font style="color:rgb(15, 17, 21);">现象</font> | <font style="color:rgb(15, 17, 21);">教训</font> |
| --- | --- | --- |
| <font style="color:rgb(15, 17, 21);">AP接口配成Access</font> | <font style="color:rgb(15, 17, 21);">STA拿到管理VLAN IP</font> | <font style="color:rgb(15, 17, 21);">直接转发下AP接口应为Trunk，PVID=管理VLAN，放行业务VLAN。</font> |
| <font style="color:rgb(15, 17, 21);">R1多配</font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">ppp authentication-mode chap</font>` | <font style="color:rgb(15, 17, 21);">PPPoE拨号失败（LCP协商失败）</font> | <font style="color:rgb(15, 17, 21);">客户端不应配置认证方式。</font> |
| ~~<font style="color:rgb(15, 17, 21);">缺少</font>~~`~~<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">dialer-rule</font>~~` | ~~<font style="color:rgb(15, 17, 21);">物理接口不触发拨号</font>~~ | ~~<font style="color:rgb(15, 17, 21);">必须配置拨号规则并绑定。</font>~~ |
| ~~<font style="color:rgb(15, 17, 21);">R2缺少</font>~~`~~<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">peer default ip address pool(思科命令)</font>~~` | ~~<font style="color:rgb(15, 17, 21);">R1拿不到IP</font>~~ | ~~<font style="color:rgb(15, 17, 21);">服务端必须指定地址池。</font>~~ |
| <font style="color:rgb(15, 17, 21);">ACL写错方向或源网段</font> | <font style="color:rgb(15, 17, 21);">VLAN20仍可访问HTTP</font> | <font style="color:rgb(15, 17, 21);">仔细核对需求：</font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">deny tcp source 192.168.20.0</font>`<br/><font style="color:rgb(15, 17, 21);">。</font> |
| <font style="color:rgb(15, 17, 21);">Trunk未放行所有业务VLAN</font> | <font style="color:rgb(15, 17, 21);">跨交换机二层不通</font> | <font style="color:rgb(15, 17, 21);">按需求“放行除VLAN1外的所有业务VLAN”。</font> |
| <font style="color:rgb(15, 17, 21);">DHCP未配置</font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">dns-list</font>` | <font style="color:rgb(15, 17, 21);">PC无法解析域名</font> | <font style="color:rgb(15, 17, 21);">DHCP必须下放DNS。</font> |


---

## <font style="color:rgb(15, 17, 21);">五、验证命令集（按模块）</font>
### <font style="color:rgb(15, 17, 21);">内网二层</font>
<font style="color:rgb(15, 17, 21);">bash</font>

```plain
display vlan
display interface trunk
display stp root
```

### <font style="color:rgb(15, 17, 21);">内网三层（OSPF 1）</font>
<font style="color:rgb(15, 17, 21);">bash</font>

```plain
display ospf peer brief
display ip routing-table
ping 192.168.30.2   # PC1 ping PC3
```

### <font style="color:rgb(15, 17, 21);">无线</font>
<font style="color:rgb(15, 17, 21);">bash</font>

```plain
display ap all
display vap all
display station all
```

### <font style="color:rgb(15, 17, 21);">PPPoE与NAT</font>
<font style="color:rgb(15, 17, 21);">bash</font>

```plain
display interface Dialer1
display pppoe-client session summary
display nat outbound
display nat session all
```

### <font style="color:rgb(15, 17, 21);">ISP侧（OSPF 2）</font>
<font style="color:rgb(15, 17, 21);">bash</font>

```plain
display ospf peer brief
display ip routing-table
ping 3.0.0.100
```

### <font style="color:rgb(15, 17, 21);">业务验证</font>
<font style="color:rgb(15, 17, 21);">bash</font>

```plain
# PC1
ping 3.0.0.100
ping www.wakin.com
# PC2
浏览器访问 http://4.0.0.100   # 应被拒绝
# PC3（VLAN30）正常上网
ping 4.0.0.100
```

---

## <font style="color:rgb(15, 17, 21);">六、实验收获</font>
1. **<font style="color:rgb(15, 17, 21);">理解了三层交换与VLAN间路由的本质</font>**<font style="color:rgb(15, 17, 21);">：SVI接口终结VLAN标签，路由后再封装。</font>
2. **<font style="color:rgb(15, 17, 21);">掌握了直接转发模式下AP接口的正确配置</font>**<font style="color:rgb(15, 17, 21);">：Trunk + PVID管理VLAN + 放行业务VLAN。</font>
3. **<font style="color:rgb(15, 17, 21);">熟悉了PPPoE拨号的完整流程</font>**<font style="color:rgb(15, 17, 21);">：客户端需配置</font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">dialer-rule</font>`<font style="color:rgb(15, 17, 21);">和</font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">dialer-group</font>`<font style="color:rgb(15, 17, 21);">，服务端需配置</font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">peer default ip address pool</font>`<font style="color:rgb(15, 17, 21);">。</font>
4. **<font style="color:rgb(15, 17, 21);">学会了NAT/PAT与ACL的配合</font>**<font style="color:rgb(15, 17, 21);">：先定义ACL，再在接口下调用</font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">nat outbound</font>`<font style="color:rgb(15, 17, 21);">。</font>
5. **<font style="color:rgb(15, 17, 21);">提升了排错能力</font>**<font style="color:rgb(15, 17, 21);">：通过日志（</font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">display logbuffer</font>`<font style="color:rgb(15, 17, 21);">）、路由表、OSPF邻居状态、抓包等多维度定位问题。</font>

---

## <font style="color:rgb(15, 17, 21);">七、后续建议</font>
+ <font style="color:rgb(15, 17, 21);">可进一步学习</font>**<font style="color:rgb(15, 17, 21);">IPsec VPN</font>**<font style="color:rgb(15, 17, 21);">，将分支与总部通过公网安全连接。</font>
+ <font style="color:rgb(15, 17, 21);">可研究</font>**<font style="color:rgb(15, 17, 21);">DHCP中继</font>**<font style="color:rgb(15, 17, 21);">，当DHCP服务器与客户端不在同一网段时的配置。</font>
+ <font style="color:rgb(15, 17, 21);">可尝试</font>**<font style="color:rgb(15, 17, 21);">IPv6</font>**<font style="color:rgb(15, 17, 21);">过渡技术，如双栈或6to4隧道。</font>

# <font style="color:rgb(15, 17, 21);">一、SW1（接入交换机）</font>
## <font style="color:rgb(15, 17, 21);">配置单</font>
| <font style="color:rgb(15, 17, 21);">配置项</font> | <font style="color:rgb(15, 17, 21);">内容</font> |
| --- | --- |
| <font style="color:rgb(15, 17, 21);">主机名</font> | <font style="color:rgb(15, 17, 21);">SW1</font> |
| <font style="color:rgb(15, 17, 21);">VLAN</font> | <font style="color:rgb(15, 17, 21);">10、20、50、60、100</font> |
| <font style="color:rgb(15, 17, 21);">G0/0/1</font> | <font style="color:rgb(15, 17, 21);">Access，VLAN 10（PC1）</font> |
| <font style="color:rgb(15, 17, 21);">G0/0/2</font> | <font style="color:rgb(15, 17, 21);">Access，VLAN 20（PC2）</font> |
| <font style="color:rgb(15, 17, 21);">G0/0/20</font> | <font style="color:rgb(15, 17, 21);">Trunk，PVID 100，放行 VLAN 50、100，禁止 VLAN 1（连接AP1）</font> |
| <font style="color:rgb(15, 17, 21);">G0/0/23</font> | <font style="color:rgb(15, 17, 21);">Trunk，放行 VLAN 10、20、30、40、50、60、100，禁止 VLAN 1（上行SW3）</font> |
| <font style="color:rgb(15, 17, 21);">G0/0/24</font> | <font style="color:rgb(15, 17, 21);">Trunk，放行 VLAN 10、20、30、40、50、60、100，禁止 VLAN 1（连接SW2）</font> |


## <font style="color:rgb(15, 17, 21);">命令组</font>
<font style="color:rgb(15, 17, 21);">bash</font>

```plain
system-view
sysname SW1
vlan batch 10 20 50 60 100
interface GigabitEthernet0/0/1
 port link-type access
 port default vlan 10
interface GigabitEthernet0/0/2
 port link-type access
 port default vlan 20
interface GigabitEthernet0/0/20
 port link-type trunk
 port trunk pvid vlan 100
 port trunk allow-pass vlan 50 100
 undo port trunk allow-pass vlan 1
interface GigabitEthernet0/0/23
 port link-type trunk
 port trunk allow-pass vlan 10 20 30 40 50 60 100
 undo port trunk allow-pass vlan 1
interface GigabitEthernet0/0/24
 port link-type trunk
 port trunk allow-pass vlan 10 20 30 40 50 60 100
 undo port trunk allow-pass vlan 1
quit
```

---

# <font style="color:rgb(15, 17, 21);">二、SW2（接入交换机）</font>
## <font style="color:rgb(15, 17, 21);">配置单</font>
| <font style="color:rgb(15, 17, 21);">配置项</font> | <font style="color:rgb(15, 17, 21);">内容</font> |
| --- | --- |
| <font style="color:rgb(15, 17, 21);">主机名</font> | <font style="color:rgb(15, 17, 21);">SW2</font> |
| <font style="color:rgb(15, 17, 21);">VLAN</font> | <font style="color:rgb(15, 17, 21);">30、40、50、60、100</font> |
| <font style="color:rgb(15, 17, 21);">G0/0/1</font> | <font style="color:rgb(15, 17, 21);">Access，VLAN 30（PC3）</font> |
| <font style="color:rgb(15, 17, 21);">G0/0/2</font> | <font style="color:rgb(15, 17, 21);">Access，VLAN 40（PC4）</font> |
| <font style="color:rgb(15, 17, 21);">G0/0/20</font> | <font style="color:rgb(15, 17, 21);">Trunk，PVID 100，放行 VLAN 60、100，禁止 VLAN 1（连接AP2）</font> |
| <font style="color:rgb(15, 17, 21);">G0/0/23</font> | <font style="color:rgb(15, 17, 21);">Trunk，放行 VLAN 10、20、30、40、50、60、100，禁止 VLAN 1（上行SW3）</font> |
| <font style="color:rgb(15, 17, 21);">G0/0/24</font> | <font style="color:rgb(15, 17, 21);">Trunk，放行 VLAN 10、20、30、40、50、60、100，禁止 VLAN 1（连接SW1）</font> |


## <font style="color:rgb(15, 17, 21);">命令组</font>
<font style="color:rgb(15, 17, 21);">bash</font>

```plain
system-view
sysname SW2
vlan batch 30 40 50 60 100
interface GigabitEthernet0/0/1
 port link-type access
 port default vlan 30
interface GigabitEthernet0/0/2
 port link-type access
 port default vlan 40
interface GigabitEthernet0/0/20
 port link-type trunk
 port trunk pvid vlan 100
 port trunk allow-pass vlan 60 100
 undo port trunk allow-pass vlan 1
interface GigabitEthernet0/0/23
 port link-type trunk
 port trunk allow-pass vlan 10 20 30 40 50 60 100
 undo port trunk allow-pass vlan 1
interface GigabitEthernet0/0/24
 port link-type trunk
 port trunk allow-pass vlan 10 20 30 40 50 60 100
 undo port trunk allow-pass vlan 1
quit
```

---

# <font style="color:rgb(15, 17, 21);">三、SW3（核心交换机）</font>
## <font style="color:rgb(15, 17, 21);">配置单</font>
| <font style="color:rgb(15, 17, 21);">配置项</font> | <font style="color:rgb(15, 17, 21);">内容</font> |
| --- | --- |
| <font style="color:rgb(15, 17, 21);">主机名</font> | <font style="color:rgb(15, 17, 21);">SW3</font> |
| <font style="color:rgb(15, 17, 21);">VLAN</font> | <font style="color:rgb(15, 17, 21);">10、20、30、40、50、60、35、51、100</font> |
| <font style="color:rgb(15, 17, 21);">STP优先级</font> | <font style="color:rgb(15, 17, 21);">4096（根桥）</font> |
| <font style="color:rgb(15, 17, 21);">DHCP</font> | <font style="color:rgb(15, 17, 21);">全局启用</font> |
| <font style="color:rgb(15, 17, 21);">Vlanif10</font> | <font style="color:rgb(15, 17, 21);">IP 192.168.10.1/24，DHCP接口模式，DNS 3.0.0.100</font> |
| <font style="color:rgb(15, 17, 21);">Vlanif20</font> | <font style="color:rgb(15, 17, 21);">IP 192.168.20.1/24，DHCP接口模式，DNS 3.0.0.100</font> |
| <font style="color:rgb(15, 17, 21);">Vlanif30</font> | <font style="color:rgb(15, 17, 21);">IP 192.168.30.1/24，DHCP接口模式，DNS 3.0.0.100</font> |
| <font style="color:rgb(15, 17, 21);">Vlanif40</font> | <font style="color:rgb(15, 17, 21);">IP 192.168.40.1/24，DHCP接口模式，DNS 3.0.0.100</font> |
| <font style="color:rgb(15, 17, 21);">Vlanif50</font> | <font style="color:rgb(15, 17, 21);">IP 192.168.50.1/24，DHCP接口模式，DNS 3.0.0.100</font> |
| <font style="color:rgb(15, 17, 21);">Vlanif60</font> | <font style="color:rgb(15, 17, 21);">IP 192.168.60.1/24，DHCP接口模式，DNS 3.0.0.100</font> |
| <font style="color:rgb(15, 17, 21);">Vlanif35</font> | <font style="color:rgb(15, 17, 21);">IP 192.168.35.3/24</font> |
| <font style="color:rgb(15, 17, 21);">Vlanif51</font> | <font style="color:rgb(15, 17, 21);">IP 192.168.51.3/24</font> |
| <font style="color:rgb(15, 17, 21);">Vlanif100</font> | <font style="color:rgb(15, 17, 21);">IP 192.168.100.1/24，DHCP接口模式，Option 43 sub-2：192.168.51.1</font> |
| <font style="color:rgb(15, 17, 21);">G0/0/1</font> | <font style="color:rgb(15, 17, 21);">Trunk，放行 VLAN 10、20、30、40、50、60、100，禁止 VLAN 1</font> |
| <font style="color:rgb(15, 17, 21);">G0/0/2</font> | <font style="color:rgb(15, 17, 21);">Trunk，放行 VLAN 10、20、30、40、50、60、100，禁止 VLAN 1</font> |
| <font style="color:rgb(15, 17, 21);">Eth-Trunk1</font> | <font style="color:rgb(15, 17, 21);">LACP静态，最大活动链路2，成员 G0/0/21-23，Trunk放行 VLAN 35，PVID 35</font> |
| <font style="color:rgb(15, 17, 21);">OSPF 1</font> | <font style="color:rgb(15, 17, 21);">Router-id 3.3.3.3，Area 0，宣告所有192.168.x.0/24网段</font> |


## <font style="color:rgb(15, 17, 21);">命令组</font>
<font style="color:rgb(15, 17, 21);">bash</font>

```plain
system-view
sysname SW3
vlan batch 10 20 30 40 50 60 35 51 100
stp priority 4096
dhcp enable
interface Vlanif10
 ip address 192.168.10.1 255.255.255.0
 dhcp select interface
 dhcp server dns-list 3.0.0.100
interface Vlanif20
 ip address 192.168.20.1 255.255.255.0
 dhcp select interface
 dhcp server dns-list 3.0.0.100
interface Vlanif30
 ip address 192.168.30.1 255.255.255.0
 dhcp select interface
 dhcp server dns-list 3.0.0.100
interface Vlanif40
 ip address 192.168.40.1 255.255.255.0
 dhcp select interface
 dhcp server dns-list 3.0.0.100
interface Vlanif50
 ip address 192.168.50.1 255.255.255.0
 dhcp select interface
 dhcp server dns-list 3.0.0.100
interface Vlanif60
 ip address 192.168.60.1 255.255.255.0
 dhcp select interface
 dhcp server dns-list 3.0.0.100
interface Vlanif35
 ip address 192.168.35.3 255.255.255.0
interface Vlanif51
 ip address 192.168.51.3 255.255.255.0
interface Vlanif100
 ip address 192.168.100.1 255.255.255.0
 dhcp select interface
 dhcp server option 43 sub-option 2 ip-address 192.168.51.1
interface GigabitEthernet0/0/1
 port link-type trunk
 port trunk allow-pass vlan 10 20 30 40 50 60 100
 undo port trunk allow-pass vlan 1
interface GigabitEthernet0/0/2
 port link-type trunk
 port trunk allow-pass vlan 10 20 30 40 50 60 100
 undo port trunk allow-pass vlan 1
interface Eth-Trunk1
 mode lacp-static
 max active-linknumber 2
 trunkport GigabitEthernet 0/0/21 to 0/0/23
 port link-type trunk
 port trunk allow-pass vlan 35
 undo port trunk allow-pass vlan 1
 port trunk pvid vlan 35
ospf 1 router-id 3.3.3.3
 area 0.0.0.0
  network 192.168.10.0 0.0.0.255
  network 192.168.20.0 0.0.0.255
  network 192.168.30.0 0.0.0.255
  network 192.168.40.0 0.0.0.255
  network 192.168.50.0 0.0.0.255
  network 192.168.60.0 0.0.0.255
  network 192.168.35.0 0.0.0.255
  network 192.168.51.0 0.0.0.255
  network 192.168.100.0 0.0.0.255
quit
```

---

# <font style="color:rgb(15, 17, 21);">四、SW5（汇聚交换机）</font>
## <font style="color:rgb(15, 17, 21);">配置单</font>
| <font style="color:rgb(15, 17, 21);">配置项</font> | <font style="color:rgb(15, 17, 21);">内容</font> |
| --- | --- |
| <font style="color:rgb(15, 17, 21);">主机名</font> | <font style="color:rgb(15, 17, 21);">SW5</font> |
| <font style="color:rgb(15, 17, 21);">LACP优先级</font> | <font style="color:rgb(15, 17, 21);">1（主动端）</font> |
| <font style="color:rgb(15, 17, 21);">VLAN</font> | <font style="color:rgb(15, 17, 21);">35、15、51、100</font> |
| <font style="color:rgb(15, 17, 21);">Vlanif35</font> | <font style="color:rgb(15, 17, 21);">IP 192.168.35.5/24</font> |
| <font style="color:rgb(15, 17, 21);">Vlanif15</font> | <font style="color:rgb(15, 17, 21);">IP 192.168.15.5/24</font> |
| <font style="color:rgb(15, 17, 21);">Vlanif51</font> | <font style="color:rgb(15, 17, 21);">IP 192.168.51.5/24</font> |
| <font style="color:rgb(15, 17, 21);">Vlanif100</font> | <font style="color:rgb(15, 17, 21);">IP 192.168.100.5/24</font> |
| <font style="color:rgb(15, 17, 21);">Eth-Trunk1</font> | <font style="color:rgb(15, 17, 21);">LACP静态，最大活动链路2，成员 G0/0/21-23，Trunk放行 VLAN 35，PVID 35</font> |
| <font style="color:rgb(15, 17, 21);">G0/0/1</font> | <font style="color:rgb(15, 17, 21);">Access，VLAN 15（连接R1）</font> |
| <font style="color:rgb(15, 17, 21);">G0/0/24</font> | <font style="color:rgb(15, 17, 21);">Access，VLAN 51（连接AC1）</font> |
| <font style="color:rgb(15, 17, 21);">OSPF 1</font> | <font style="color:rgb(15, 17, 21);">Router-id 5.5.5.5，Area 0，宣告 192.168.35.0/24、192.168.15.0/24、192.168.51.0/24、192.168.100.0/24</font> |


## <font style="color:rgb(15, 17, 21);">命令组</font>
<font style="color:rgb(15, 17, 21);">bash</font>

```plain
system-view
sysname SW5
lacp priority 1
vlan batch 35 15 51 100
interface Vlanif35
 ip address 192.168.35.5 255.255.255.0
interface Vlanif15
 ip address 192.168.15.5 255.255.255.0
interface Vlanif51
 ip address 192.168.51.5 255.255.255.0
interface Vlanif100
 ip address 192.168.100.5 255.255.255.0
interface Eth-Trunk1
 mode lacp-static
 max active-linknumber 2
 trunkport GigabitEthernet 0/0/21 to 0/0/23
 port link-type trunk
 port trunk allow-pass vlan 35
 undo port trunk allow-pass vlan 1
 port trunk pvid vlan 35
interface GigabitEthernet0/0/1
 port link-type access
 port default vlan 15
 undo shutdown
interface GigabitEthernet0/0/24
 port link-type access
 port default vlan 51
ospf 1 router-id 5.5.5.5
 area 0.0.0.0
  network 192.168.35.0 0.0.0.255
  network 192.168.15.0 0.0.0.255
  network 192.168.51.0 0.0.0.255
  network 192.168.100.0 0.0.0.255
quit
```

---

# <font style="color:rgb(15, 17, 21);">五、AC1（无线控制器）</font>
## <font style="color:rgb(15, 17, 21);">配置单</font>
| <font style="color:rgb(15, 17, 21);">配置项</font> | <font style="color:rgb(15, 17, 21);">内容</font> |
| --- | --- |
| <font style="color:rgb(15, 17, 21);">主机名</font> | <font style="color:rgb(15, 17, 21);">AC1</font> |
| <font style="color:rgb(15, 17, 21);">VLAN</font> | <font style="color:rgb(15, 17, 21);">51</font> |
| <font style="color:rgb(15, 17, 21);">G0/0/1</font> | <font style="color:rgb(15, 17, 21);">Access，VLAN 51</font> |
| <font style="color:rgb(15, 17, 21);">Vlanif51</font> | <font style="color:rgb(15, 17, 21);">IP 192.168.51.1/24</font> |
| <font style="color:rgb(15, 17, 21);">CAPWAP源接口</font> | <font style="color:rgb(15, 17, 21);">Vlanif51</font> |
| <font style="color:rgb(15, 17, 21);">OSPF 1</font> | <font style="color:rgb(15, 17, 21);">Router-id 192.168.51.1，Area 0，宣告 192.168.51.0/24</font> |
| <font style="color:rgb(15, 17, 21);">域管理模板</font> | <font style="color:rgb(15, 17, 21);">HCIA，国家码 CN</font> |
| <font style="color:rgb(15, 17, 21);">AP组 Office</font> | <font style="color:rgb(15, 17, 21);">绑定域管理模板 HCIA，绑定VAP模板 Office</font> |
| <font style="color:rgb(15, 17, 21);">AP组 Guest</font> | <font style="color:rgb(15, 17, 21);">绑定域管理模板 HCIA，绑定VAP模板 Guest</font> |
| <font style="color:rgb(15, 17, 21);">SSID模板 Office</font> | <font style="color:rgb(15, 17, 21);">SSID：Office</font> |
| <font style="color:rgb(15, 17, 21);">SSID模板 Guest</font> | <font style="color:rgb(15, 17, 21);">SSID：Guest</font> |
| <font style="color:rgb(15, 17, 21);">安全模板 Office</font> | <font style="color:rgb(15, 17, 21);">WPA2-PSK，密码 12345678，AES</font> |
| <font style="color:rgb(15, 17, 21);">安全模板 Guest</font> | <font style="color:rgb(15, 17, 21);">WPA2-PSK，密码 12345678，AES</font> |
| <font style="color:rgb(15, 17, 21);">VAP模板 Office</font> | <font style="color:rgb(15, 17, 21);">直接转发，业务VLAN 50，绑定SSID Office，绑定安全模板 Office</font> |
| <font style="color:rgb(15, 17, 21);">VAP模板 Guest</font> | <font style="color:rgb(15, 17, 21);">直接转发，业务VLAN 60，绑定SSID Guest，绑定安全模板 Guest</font> |
| <font style="color:rgb(15, 17, 21);">AP 1</font> | <font style="color:rgb(15, 17, 21);">ID 1，MAC 00e0-fcad-5ea0，名称 Office-AP1，分组 Office</font> |
| <font style="color:rgb(15, 17, 21);">AP 2</font> | <font style="color:rgb(15, 17, 21);">ID 2，MAC 00e0-fc68-5780，名称 Guest-AP1，分组 Guest</font> |


## <font style="color:rgb(15, 17, 21);">命令组</font>
<font style="color:rgb(15, 17, 21);">bash</font>

```plain
system-view
sysname AC1
vlan batch 51
interface GigabitEthernet0/0/1
 port link-type access
 port default vlan 51
interface Vlanif51
 ip address 192.168.51.1 255.255.255.0
ospf 1 router-id 192.168.51.1
 area 0.0.0.0
  network 192.168.51.0 0.0.0.255
capwap source interface vlanif 51
wlan
 regulatory-domain-profile name HCIA
  country-code CN
 ap-group name Office
  regulatory-domain-profile HCIA
 ap-group name Guest
  regulatory-domain-profile HCIA
 ssid-profile name Office
  ssid Office
 ssid-profile name Guest
  ssid Guest
 security-profile name Office
  security wpa2 psk pass-phrase 12345678 aes
 security-profile name Guest
  security wpa2 psk pass-phrase 12345678 aes
 vap-profile name Office
  forward-mode direct-forward
  service-vlan vlan-id 50
  ssid-profile Office
  security-profile Office
 vap-profile name Guest
  forward-mode direct-forward
  service-vlan vlan-id 60
  ssid-profile Guest
  security-profile Guest
 ap-group name Office
  vap-profile Office wlan 1 radio all
 ap-group name Guest
  vap-profile Guest wlan 1 radio all
 ap-id 1 ap-mac 00e0-fcad-5ea0
  ap-name Office-AP1
  ap-group Office
 ap-id 2 ap-mac 00e0-fc68-5780
  ap-name Guest-AP1
  ap-group Guest
quit
```

---

# <font style="color:rgb(15, 17, 21);">六、R1（出口路由器）</font>
## <font style="color:rgb(15, 17, 21);">配置单</font>
| <font style="color:rgb(15, 17, 21);">配置项</font> | <font style="color:rgb(15, 17, 21);">内容</font> |
| --- | --- |
| <font style="color:rgb(15, 17, 21);">主机名</font> | <font style="color:rgb(15, 17, 21);">R1</font> |
| <font style="color:rgb(15, 17, 21);">G0/0/1</font> | <font style="color:rgb(15, 17, 21);">IP 192.168.15.1/24</font> |
| <font style="color:rgb(15, 17, 21);">ACL 3000</font> | <font style="color:rgb(15, 17, 21);">rule 5 deny tcp source 192.168.20.0 0.0.0.255 destination-port eq 80</font> |
| <font style="color:rgb(15, 17, 21);">ACL 3000</font> | <font style="color:rgb(15, 17, 21);">rule 10 permit ip</font> |
| <font style="color:rgb(15, 17, 21);">G0/0/2</font> | <font style="color:rgb(15, 17, 21);">PPPoE客户端，绑定 Dialer1</font> |
| <font style="color:rgb(15, 17, 21);">Dialer1</font> | <font style="color:rgb(15, 17, 21);">PPP CHAP，用户名 wakin，密码</font><font style="color:rgb(15, 17, 21);"> </font>[<font style="color:rgb(57, 100, 254);">666.com</font>](https://666.com/)<br/><font style="color:rgb(15, 17, 21);">，IP地址协商，MTU</font><font style="color:rgb(15, 17, 21);"> </font><font style="color:rgb(15, 17, 21);">1492，TCP MSS 1452，NAT outbound 3000，dialer-group 1</font> |
| <font style="color:rgb(15, 17, 21);">dialer-rule</font> | <font style="color:rgb(15, 17, 21);">dialer-rule 1 ip permit</font> |
| <font style="color:rgb(15, 17, 21);">静态缺省路由</font> | <font style="color:rgb(15, 17, 21);">0.0.0.0 0.0.0.0 Dialer1</font> |
| <font style="color:rgb(15, 17, 21);">OSPF 1</font> | <font style="color:rgb(15, 17, 21);">Router-id 1.1.1.1，Area 0，宣告 192.168.15.0/24，default-route-advertise always</font> |
| <font style="color:rgb(15, 17, 21);">静态路由（可选）</font> | <font style="color:rgb(15, 17, 21);">ip route-static 3.0.0.0 255.255.255.0 Dialer1（若OSPF未学到）</font> |


## <font style="color:rgb(15, 17, 21);">命令组</font>
<font style="color:rgb(15, 17, 21);">bash</font>

```plain
system-view
sysname R1
interface GigabitEthernet0/0/1
 ip address 192.168.15.1 255.255.255.0
 undo shutdown
acl number 3000
 rule 5 deny tcp source 192.168.20.0 0.0.0.255 destination-port eq 80
 rule 10 permit ip
interface GigabitEthernet0/0/2
 pppoe-client dial-bundle-number 1
dialer-rule
 dialer-rule 1 ip permit
interface Dialer1
 ppp authentication-mode chap
 link-protocol ppp
 ppp chap user wakin
 ppp chap password cipher 666.com
 ppp ipcp default-route
 ip address ppp-negotiate
 dialer user wakin
 dialer bundle 1
 mtu 1492
 tcp mss 1452
 nat outbound 3000
 dialer-group 1
ip route-static 0.0.0.0 0.0.0.0 Dialer1
ospf 1 router-id 1.1.1.1
 area 0.0.0.0
  network 192.168.15.0 0.0.0.255
 default-route-advertise always
quit
```

---

# <font style="color:rgb(15, 17, 21);">七、R2（PPPoE服务器 + OSPF 2）</font>
## <font style="color:rgb(15, 17, 21);">配置单</font>
| <font style="color:rgb(15, 17, 21);">配置项</font> | <font style="color:rgb(15, 17, 21);">内容</font> |
| --- | --- |
| <font style="color:rgb(15, 17, 21);">主机名</font> | <font style="color:rgb(15, 17, 21);">R2</font> |
| <font style="color:rgb(15, 17, 21);">Loopback0</font> | <font style="color:rgb(15, 17, 21);">IP 2.2.2.2/32</font> |
| <font style="color:rgb(15, 17, 21);">AAA</font> | <font style="color:rgb(15, 17, 21);">本地用户 wakin，密码</font><font style="color:rgb(15, 17, 21);"> </font>[<font style="color:rgb(57, 100, 254);">666.com</font>](https://666.com/)<br/><font style="color:rgb(15, 17, 21);">，服务类型</font><font style="color:rgb(15, 17, 21);"> </font><font style="color:rgb(15, 17, 21);">ppp</font> |
| <font style="color:rgb(15, 17, 21);">地址池 pppoe</font> | <font style="color:rgb(15, 17, 21);">网关 66.66.66.1，网段 66.66.66.0/24，地址范围 66.66.66.10 - 66.66.66.200</font> |
| <font style="color:rgb(15, 17, 21);">Virtual-Template 1</font> | <font style="color:rgb(15, 17, 21);">PPP认证方式 CHAP，地址池 pppoe，peer default ip address pool pppoe，IP 66.66.66.1/24</font> |
| <font style="color:rgb(15, 17, 21);">G0/0/2</font> | <font style="color:rgb(15, 17, 21);">绑定 Virtual-Template 1（pppoe-server）</font> |
| <font style="color:rgb(15, 17, 21);">G0/0/0</font> | <font style="color:rgb(15, 17, 21);">IP 23.0.0.1/24</font> |
| <font style="color:rgb(15, 17, 21);">G0/0/1</font> | <font style="color:rgb(15, 17, 21);">IP 24.0.0.1/24</font> |
| <font style="color:rgb(15, 17, 21);">OSPF 2</font> | <font style="color:rgb(15, 17, 21);">Router-id 2.2.2.2，Area 0，发布 66.66.66.0/24、23.0.0.0/24、24.0.0.0/24</font> |


## <font style="color:rgb(15, 17, 21);">命令组</font>
<font style="color:rgb(15, 17, 21);">bash</font>

```plain
system-view
sysname R2
interface Loopback0
 ip address 2.2.2.2 255.255.255.255
ip pool pppoe
 gateway 66.66.66.1
 network 66.66.66.0 mask 255.255.255.0
 section 0 66.66.66.10 66.66.66.200
interface Virtual-Template1
 ppp authentication-mode chap
 remote address pool pppoe
 peer default ip address pool pppoe
 ip address 66.66.66.1 255.255.255.0
interface GigabitEthernet0/0/2
 pppoe-server bind virtual-template 1
interface GigabitEthernet0/0/0
 ip address 23.0.0.1 255.255.255.0
interface GigabitEthernet0/0/1
 ip address 24.0.0.1 255.255.255.0
aaa
 local-user wakin password cipher 666.com
 local-user wakin service-type ppp
ospf 2 router-id 2.2.2.2
 area 0.0.0.0
  network 66.66.66.0 0.0.0.255
  network 23.0.0.0 0.0.0.255
  network 24.0.0.0 0.0.0.255
quit
```

---

# <font style="color:rgb(15, 17, 21);">八、R3（ISP路由器）</font>
## <font style="color:rgb(15, 17, 21);">配置单</font>
| <font style="color:rgb(15, 17, 21);">配置项</font> | <font style="color:rgb(15, 17, 21);">内容</font> |
| --- | --- |
| <font style="color:rgb(15, 17, 21);">主机名</font> | <font style="color:rgb(15, 17, 21);">R3</font> |
| <font style="color:rgb(15, 17, 21);">G0/0/0</font> | <font style="color:rgb(15, 17, 21);">IP 23.0.0.2/24</font> |
| <font style="color:rgb(15, 17, 21);">G0/0/1</font> | <font style="color:rgb(15, 17, 21);">IP 34.0.0.1/24</font> |
| <font style="color:rgb(15, 17, 21);">G0/0/2</font> | <font style="color:rgb(15, 17, 21);">IP 3.0.0.1/24</font> |
| <font style="color:rgb(15, 17, 21);">OSPF 2</font> | <font style="color:rgb(15, 17, 21);">Router-id 3.3.3.3，Area 0，发布 23.0.0.0/24、34.0.0.0/24、3.0.0.0/24</font> |


## <font style="color:rgb(15, 17, 21);">命令组</font>
<font style="color:rgb(15, 17, 21);">bash</font>

```plain
system-view
sysname R3
interface GigabitEthernet0/0/0
 ip address 23.0.0.2 255.255.255.0
interface GigabitEthernet0/0/1
 ip address 34.0.0.1 255.255.255.0
interface GigabitEthernet0/0/2
 ip address 3.0.0.1 255.255.255.0
ospf 2 router-id 3.3.3.3
 area 0.0.0.0
  network 23.0.0.0 0.0.0.255
  network 34.0.0.0 0.0.0.255
  network 3.0.0.0 0.0.0.255
quit
```

---

# <font style="color:rgb(15, 17, 21);">九、R4（ISP路由器）</font>
## <font style="color:rgb(15, 17, 21);">配置单</font>
| <font style="color:rgb(15, 17, 21);">配置项</font> | <font style="color:rgb(15, 17, 21);">内容</font> |
| --- | --- |
| <font style="color:rgb(15, 17, 21);">主机名</font> | <font style="color:rgb(15, 17, 21);">R4</font> |
| <font style="color:rgb(15, 17, 21);">G0/0/0</font> | <font style="color:rgb(15, 17, 21);">IP 24.0.0.2/24</font> |
| <font style="color:rgb(15, 17, 21);">G0/0/1</font> | <font style="color:rgb(15, 17, 21);">IP 34.0.0.2/24</font> |
| <font style="color:rgb(15, 17, 21);">G0/0/2</font> | <font style="color:rgb(15, 17, 21);">IP 4.0.0.1/24</font> |
| <font style="color:rgb(15, 17, 21);">OSPF 2</font> | <font style="color:rgb(15, 17, 21);">Router-id 4.4.4.4，Area 0，发布 24.0.0.0/24、34.0.0.0/24、4.0.0.0/24</font> |


## <font style="color:rgb(15, 17, 21);">命令组</font>
<font style="color:rgb(15, 17, 21);">bash</font>

```plain
system-view
sysname R4
interface GigabitEthernet0/0/0
 ip address 24.0.0.2 255.255.255.0
interface GigabitEthernet0/0/1
 ip address 34.0.0.2 255.255.255.0
interface GigabitEthernet0/0/2
 ip address 4.0.0.1 255.255.255.0
ospf 2 router-id 4.4.4.4
 area 0.0.0.0
  network 24.0.0.0 0.0.0.255
  network 34.0.0.0 0.0.0.255
  network 4.0.0.0 0.0.0.255
quit
```

---

# <font style="color:rgb(15, 17, 21);">十、DNS服务器</font>
## <font style="color:rgb(15, 17, 21);">配置单</font>
| <font style="color:rgb(15, 17, 21);">配置项</font> | <font style="color:rgb(15, 17, 21);">内容</font> |
| --- | --- |
| <font style="color:rgb(15, 17, 21);">主机名</font> | <font style="color:rgb(15, 17, 21);">DNS</font> |
| <font style="color:rgb(15, 17, 21);">E0/0/0</font> | <font style="color:rgb(15, 17, 21);">IP 3.0.0.100/24</font> |
| <font style="color:rgb(15, 17, 21);">DNS服务</font> | <font style="color:rgb(15, 17, 21);">启用DNS解析，启用DNS代理，DNS服务器 8.8.8.8</font> |
| <font style="color:rgb(15, 17, 21);">静态域名</font> | [<font style="color:rgb(57, 100, 254);">www.wakin.com</font>](https://www.wakin.com/)<br/><font style="color:rgb(15, 17, 21);"> </font><font style="color:rgb(15, 17, 21);">→ 4.0.0.100</font> |
| <font style="color:rgb(15, 17, 21);">静态域名</font> | [<font style="color:rgb(57, 100, 254);">www.moive.com</font>](https://www.moive.com/)<br/><font style="color:rgb(15, 17, 21);"> </font><font style="color:rgb(15, 17, 21);">→ 4.0.0.100</font> |
| <font style="color:rgb(15, 17, 21);">默认路由</font> | <font style="color:rgb(15, 17, 21);">0.0.0.0 0.0.0.0 3.0.0.1</font> |


## <font style="color:rgb(15, 17, 21);">命令组</font>
<font style="color:rgb(15, 17, 21);">bash</font>

```plain
system-view
sysname DNS
interface Ethernet0/0/0
 ip address 3.0.0.100 255.255.255.0
dns resolve
dns proxy enable
dns server 8.8.8.8
ip host www.wakin.com 4.0.0.100
ip host www.moive.com 4.0.0.100
ip route-static 0.0.0.0 0.0.0.0 3.0.0.1
quit
```

---

# <font style="color:rgb(15, 17, 21);">十一、HTTP & FTP服务器</font>
## <font style="color:rgb(15, 17, 21);">配置单</font>
| <font style="color:rgb(15, 17, 21);">配置项</font> | <font style="color:rgb(15, 17, 21);">内容</font> |
| --- | --- |
| <font style="color:rgb(15, 17, 21);">主机名</font> | <font style="color:rgb(15, 17, 21);">Server</font> |
| <font style="color:rgb(15, 17, 21);">E0/0/0</font> | <font style="color:rgb(15, 17, 21);">IP 4.0.0.100/24</font> |
| <font style="color:rgb(15, 17, 21);">HTTP服务</font> | <font style="color:rgb(15, 17, 21);">启用</font> |
| <font style="color:rgb(15, 17, 21);">FTP服务</font> | <font style="color:rgb(15, 17, 21);">启用</font> |
| <font style="color:rgb(15, 17, 21);">AAA</font> | <font style="color:rgb(15, 17, 21);">本地用户 ftpuser，密码 ftp123，权限15，服务类型 ftp</font> |
| <font style="color:rgb(15, 17, 21);">默认路由</font> | <font style="color:rgb(15, 17, 21);">0.0.0.0 0.0.0.0 4.0.0.1</font> |


## <font style="color:rgb(15, 17, 21);">命令组</font>
<font style="color:rgb(15, 17, 21);">bash</font>

```plain
system-view
sysname Server
interface Ethernet0/0/0
 ip address 4.0.0.100 255.255.255.0
http server enable
ftp server enable
aaa
 local-user ftpuser password cipher ftp123
 local-user ftpuser privilege level 15
 local-user ftpuser service-type ftp
ip route-static 0.0.0.0 0.0.0.0 4.0.0.1
quit
```
