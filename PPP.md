<img src="https://cdn.nlark.com/yuque/0/2026/png/50508853/1776588624633-3b37e7a5-4fe5-472e-bb2f-6497abeb9238.png" width="846" title="" crop="0,0,1,1" id="u2548950a" class="ne-image">

+ **<font style="color:rgb(15, 17, 21);">拓扑</font>**<font style="color:rgb(15, 17, 21);">：AR1 与 AR2 通过</font><font style="color:rgb(15, 17, 21);"> </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">POS6/0/0</font>`<font style="color:rgb(15, 17, 21);"> </font><font style="color:rgb(15, 17, 21);">串口直连。</font>
+ **<font style="color:rgb(15, 17, 21);">认证要求</font>**<font style="color:rgb(15, 17, 21);">：</font>**<font style="color:rgb(15, 17, 21);">双向认证</font>**<font style="color:rgb(15, 17, 21);"> </font><font style="color:rgb(15, 17, 21);">— AR1 要求 AR2 通过</font><font style="color:rgb(15, 17, 21);"> </font>**<font style="color:rgb(15, 17, 21);">CHAP</font>**<font style="color:rgb(15, 17, 21);"> </font><font style="color:rgb(15, 17, 21);">认证自己；AR2 要求 AR1 通过</font><font style="color:rgb(15, 17, 21);"> </font>**<font style="color:rgb(15, 17, 21);">PAP</font>**<font style="color:rgb(15, 17, 21);"> </font><font style="color:rgb(15, 17, 21);">认证自己。</font>
+ **<font style="color:rgb(15, 17, 21);">地址分配</font>**<font style="color:rgb(15, 17, 21);">：AR1 作为地址服务端，通过 PPP 地址池</font><font style="color:rgb(15, 17, 21);"> </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">ppp</font>`<font style="color:rgb(15, 17, 21);"> </font><font style="color:rgb(15, 17, 21);">为 AR2 分配 IP（网段</font><font style="color:rgb(15, 17, 21);"> </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">9.1.1.0/24</font>`<font style="color:rgb(15, 17, 21);">）。</font>
+ **<font style="color:rgb(15, 17, 21);">路由</font>**<font style="color:rgb(15, 17, 21);">：AR2 通过</font><font style="color:rgb(15, 17, 21);"> </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">ppp ipcp default-route</font>`<font style="color:rgb(15, 17, 21);"> </font><font style="color:rgb(15, 17, 21);">自动获取缺省路由。</font>
+ **<font style="color:rgb(15, 17, 21);">额外配置</font>**<font style="color:rgb(15, 17, 21);">：AR1 配置 Loopback 接口</font><font style="color:rgb(15, 17, 21);"> </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">1.1.1.1/32</font>`<font style="color:rgb(15, 17, 21);">（图中写</font><font style="color:rgb(15, 17, 21);"> </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">intlol</font>`<font style="color:rgb(15, 17, 21);">，应为</font><font style="color:rgb(15, 17, 21);"> </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">int LoopBack1</font>`<font style="color:rgb(15, 17, 21);">）。</font>

---

## <font style="color:rgb(15, 17, 21);">一、接口与基础配置</font>
### <font style="color:rgb(15, 17, 21);">1.1 AR1（地址服务端，CHAP 认证方，PAP 被认证方）</font>
<font style="color:rgb(15, 17, 21);">bash</font>

```plain
sysname AR1
interface Pos6/0/0
 ip address 9.1.1.1 24          # 本端接口 IP（可选，但建议配置）
 ppp authentication-mode chap   # 要求对端（AR2）通过 CHAP 认证自己
 ppp pap local-user wakin password cipher 123   # 用于向 AR2 发起 PAP 认证（自己被认证）
 remote address pool ppp        # 为对端分配地址池
 quit
aaa
 local-user winnie password cipher 456   # 用于验证 AR2 的 CHAP 身份
 local-user winnie service-type ppp
 local-user wakin password cipher 123    # 用于 PAP 认证（当 AR1 作为被认证方时）
 local-user wakin service-type ppp
 quit
ip pool ppp
 network 9.1.1.0 mask 255.255.255.0
 gateway-list 9.1.1.254        # 可选，PPP 通常不需要显式网关
interface LoopBack1
 ip address 1.1.1.1 32
```

**<font style="color:rgb(15, 17, 21);">命令解析</font>**<font style="color:rgb(15, 17, 21);">：</font>

+ `<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">ppp authentication-mode chap</font>`<font style="color:rgb(15, 17, 21);">：本端要求对端使用 CHAP 认证自己。</font>
+ `<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">ppp pap local-user wakin password cipher 123</font>`<font style="color:rgb(15, 17, 21);">：本端作为 PAP</font><font style="color:rgb(15, 17, 21);"> </font>**<font style="color:rgb(15, 17, 21);">被认证方</font>**<font style="color:rgb(15, 17, 21);">，主动发送用户名</font><font style="color:rgb(15, 17, 21);"> </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">wakin</font>`<font style="color:rgb(15, 17, 21);"> </font><font style="color:rgb(15, 17, 21);">和密码</font><font style="color:rgb(15, 17, 21);"> </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">123</font>`<font style="color:rgb(15, 17, 21);"> </font><font style="color:rgb(15, 17, 21);">给对端。</font>
+ `<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">remote address pool ppp</font>`<font style="color:rgb(15, 17, 21);">：从地址池</font><font style="color:rgb(15, 17, 21);"> </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">ppp</font>`<font style="color:rgb(15, 17, 21);"> </font><font style="color:rgb(15, 17, 21);">中为对端分配 IP 地址（通过 IPCP 协商）。</font>
+ `<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">local-user winnie service-type ppp</font>`<font style="color:rgb(15, 17, 21);">：允许用户</font><font style="color:rgb(15, 17, 21);"> </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">winnie</font>`<font style="color:rgb(15, 17, 21);"> </font><font style="color:rgb(15, 17, 21);">用于 PPP 认证（此处为 CHAP 认证方验证 AR2 的身份）。</font>

### <font style="color:rgb(15, 17, 21);">1.2 AR2（地址客户端，PAP 认证方，CHAP 被认证方）</font>
<font style="color:rgb(15, 17, 21);">bash</font>

```plain
sysname AR2
interface Pos6/0/0
 ppp authentication-mode pap    # 要求对端（AR1）通过 PAP 认证自己
 ppp chap user winnie           # 用于向 AR1 发起 CHAP 认证（自己被认证）
 ppp chap password cipher 456   # CHAP 密码
 ip address ppp-negotiate       # 通过 IPCP 从服务端获取 IP
 ppp ipcp default-route         # 自动添加缺省路由指向对端
 quit
aaa
 local-user wakin password cipher 123   # 用于验证 AR1 的 PAP 身份
 local-user wakin service-type ppp
 local-user winnie password cipher 456  # 用于 CHAP 认证（本端被认证时，但此处多余）
 local-user winnie service-type ppp
```

**<font style="color:rgb(15, 17, 21);">命令解析</font>**<font style="color:rgb(15, 17, 21);">：</font>

+ `<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">ppp authentication-mode pap</font>`<font style="color:rgb(15, 17, 21);">：本端要求对端使用 PAP 认证自己。</font>
+ `<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">ppp chap user winnie</font>`<font style="color:rgb(15, 17, 21);">：本端作为 CHAP</font><font style="color:rgb(15, 17, 21);"> </font>**<font style="color:rgb(15, 17, 21);">被认证方</font>**<font style="color:rgb(15, 17, 21);">，主动发送用户名</font><font style="color:rgb(15, 17, 21);"> </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">winnie</font>`<font style="color:rgb(15, 17, 21);"> </font><font style="color:rgb(15, 17, 21);">给对端。</font>
+ `<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">ip address ppp-negotiate</font>`<font style="color:rgb(15, 17, 21);">：接口 IP 通过 PPP IPCP 协商获取（由服务端分配）。</font>
+ `<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">ppp ipcp default-route</font>`<font style="color:rgb(15, 17, 21);">：自动添加一条缺省路由，下一跳为对端 IP。</font>

---

## <font style="color:rgb(15, 17, 21);">二、双向认证原理与报文交互</font>
### <font style="color:rgb(15, 17, 21);">2.1 认证方向分析</font>
| <font style="color:rgb(15, 17, 21);">方向</font> | <font style="color:rgb(15, 17, 21);">认证协议</font> | <font style="color:rgb(15, 17, 21);">角色</font> | <font style="color:rgb(15, 17, 21);">凭证</font> |
| --- | --- | --- | --- |
| <font style="color:rgb(15, 17, 21);">AR1 → AR2</font> | <font style="color:rgb(15, 17, 21);">CHAP</font> | <font style="color:rgb(15, 17, 21);">AR1 是认证方，AR2 是被认证方</font> | <font style="color:rgb(15, 17, 21);">AR2 发送用户名</font><font style="color:rgb(15, 17, 21);"> </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">winnie</font>`<br/><font style="color:rgb(15, 17, 21);">，密码</font><font style="color:rgb(15, 17, 21);"> </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">456</font>`<br/><font style="color:rgb(15, 17, 21);">（哈希）</font> |
| <font style="color:rgb(15, 17, 21);">AR2 → AR1</font> | <font style="color:rgb(15, 17, 21);">PAP</font> | <font style="color:rgb(15, 17, 21);">AR2 是认证方，AR1 是被认证方</font> | <font style="color:rgb(15, 17, 21);">AR1 发送用户名</font><font style="color:rgb(15, 17, 21);"> </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">wakin</font>`<br/><font style="color:rgb(15, 17, 21);">，密码</font><font style="color:rgb(15, 17, 21);"> </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">123</font>`<br/><font style="color:rgb(15, 17, 21);">（明文）</font> |


### <font style="color:rgb(15, 17, 21);">2.2 CHAP 三次握手（以 AR1 认证 AR2 为例）</font>
1. **<font style="color:rgb(15, 17, 21);">Challenge</font>**<font style="color:rgb(15, 17, 21);">：AR1 发送随机挑战值（Challenge）给 AR2。</font>
2. **<font style="color:rgb(15, 17, 21);">Response</font>**<font style="color:rgb(15, 17, 21);">：AR2 用密码</font><font style="color:rgb(15, 17, 21);"> </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">456</font>`<font style="color:rgb(15, 17, 21);"> </font><font style="color:rgb(15, 17, 21);">和挑战值进行 MD5 哈希，连同用户名</font><font style="color:rgb(15, 17, 21);"> </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">winnie</font>`<font style="color:rgb(15, 17, 21);"> </font><font style="color:rgb(15, 17, 21);">发回。</font>
3. **<font style="color:rgb(15, 17, 21);">Success/Failure</font>**<font style="color:rgb(15, 17, 21);">：AR1 用本地存储的密码（</font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">456</font>`<font style="color:rgb(15, 17, 21);">）计算哈希，对比一致则认证成功。</font>

### <font style="color:rgb(15, 17, 21);">2.3 PAP 报文（AR2 认证 AR1）</font>
+ <font style="color:rgb(15, 17, 21);">AR1 直接发送</font><font style="color:rgb(15, 17, 21);"> </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">Authenticate-Request</font>`<font style="color:rgb(15, 17, 21);"> </font><font style="color:rgb(15, 17, 21);">报文，包含用户名</font><font style="color:rgb(15, 17, 21);"> </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">wakin</font>`<font style="color:rgb(15, 17, 21);"> </font><font style="color:rgb(15, 17, 21);">和明文密码</font><font style="color:rgb(15, 17, 21);"> </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">123</font>`<font style="color:rgb(15, 17, 21);">。</font>
+ <font style="color:rgb(15, 17, 21);">AR2 验证后回复</font><font style="color:rgb(15, 17, 21);"> </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">Authenticate-Ack</font>`<font style="color:rgb(15, 17, 21);"> </font><font style="color:rgb(15, 17, 21);">或</font><font style="color:rgb(15, 17, 21);"> </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">Authenticate-Nak</font>`<font style="color:rgb(15, 17, 21);">。</font>

**<font style="color:rgb(15, 17, 21);">抓包验证</font>**<font style="color:rgb(15, 17, 21);">：在 POS 链路上抓 PPP 报文，过滤</font><font style="color:rgb(15, 17, 21);"> </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">ppp</font>`<font style="color:rgb(15, 17, 21);">，可看到 LCP 协商后，先进行 CHAP（报文类型</font><font style="color:rgb(15, 17, 21);"> </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">0xC223</font>`<font style="color:rgb(15, 17, 21);">），后进行 PAP（报文类型</font><font style="color:rgb(15, 17, 21);"> </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">0xC023</font>`<font style="color:rgb(15, 17, 21);">），顺序由配置决定。</font>

---

## <font style="color:rgb(15, 17, 21);">三、地址分配与路由</font>
### <font style="color:rgb(15, 17, 21);">3.1 IPCP 协商过程</font>
1. <font style="color:rgb(15, 17, 21);">AR2 发送</font><font style="color:rgb(15, 17, 21);"> </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">Configure-Request</font>`<font style="color:rgb(15, 17, 21);">，请求 IP 地址。</font>
2. <font style="color:rgb(15, 17, 21);">AR1 回复</font><font style="color:rgb(15, 17, 21);"> </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">Configure-Ack</font>`<font style="color:rgb(15, 17, 21);">，携带从地址池</font><font style="color:rgb(15, 17, 21);"> </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">ppp</font>`<font style="color:rgb(15, 17, 21);"> </font><font style="color:rgb(15, 17, 21);">中分配的 IP（如</font><font style="color:rgb(15, 17, 21);"> </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">9.1.1.2</font>`<font style="color:rgb(15, 17, 21);">）。</font>
3. <font style="color:rgb(15, 17, 21);">AR2 接口获得 IP</font><font style="color:rgb(15, 17, 21);"> </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">9.1.1.2/32</font>`<font style="color:rgb(15, 17, 21);">，同时生成一条主机路由指向对端。</font>

**<font style="color:rgb(15, 17, 21);">验证命令</font>**<font style="color:rgb(15, 17, 21);">：</font>

<font style="color:rgb(15, 17, 21);">bash</font>

```plain
display interface Pos6/0/0        # 查看接口 IP（应为 ppp-negotiate 获取的地址）
display ip routing-table          # 检查 AR2 是否有缺省路由指向 Dialer（或直接指向 Pos 口）
```

### <font style="color:rgb(15, 17, 21);">3.2 缺省路由</font>
+ `<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">ppp ipcp default-route</font>`<font style="color:rgb(15, 17, 21);"> </font><font style="color:rgb(15, 17, 21);">在 AR2 上自动生成</font><font style="color:rgb(15, 17, 21);"> </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">0.0.0.0/0</font>`<font style="color:rgb(15, 17, 21);"> </font><font style="color:rgb(15, 17, 21);">路由，下一跳为 AR1 的接口 IP（</font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">9.1.1.1</font>`<font style="color:rgb(15, 17, 21);">）。</font>

---

## <font style="color:rgb(15, 17, 21);">四、常见问题与排错</font>
| <font style="color:rgb(15, 17, 21);">现象</font> | <font style="color:rgb(15, 17, 21);">可能原因</font> | <font style="color:rgb(15, 17, 21);">解决方法</font> |
| --- | --- | --- |
| <font style="color:rgb(15, 17, 21);">接口物理 UP，协议 DOWN</font> | <font style="color:rgb(15, 17, 21);">LCP 协商失败（认证不匹配）</font> | <font style="color:rgb(15, 17, 21);">检查</font><font style="color:rgb(15, 17, 21);"> </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">ppp authentication-mode</font>`<br/><font style="color:rgb(15, 17, 21);"> </font><font style="color:rgb(15, 17, 21);">是否一致（双向可不同，但需各自正确）</font> |
| <font style="color:rgb(15, 17, 21);">CHAP 认证失败</font> | <font style="color:rgb(15, 17, 21);">用户名或密码错误，或</font><font style="color:rgb(15, 17, 21);"> </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">service-type</font>`<br/><font style="color:rgb(15, 17, 21);"> </font><font style="color:rgb(15, 17, 21);">未配置</font> | <font style="color:rgb(15, 17, 21);">核对</font><font style="color:rgb(15, 17, 21);"> </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">local-user</font>`<br/><font style="color:rgb(15, 17, 21);"> </font><font style="color:rgb(15, 17, 21);">用户名和密码，确保</font><font style="color:rgb(15, 17, 21);"> </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">service-type ppp</font>` |
| <font style="color:rgb(15, 17, 21);">PAP 认证失败</font> | <font style="color:rgb(15, 17, 21);">用户名密码明文不匹配</font> | <font style="color:rgb(15, 17, 21);">检查</font><font style="color:rgb(15, 17, 21);"> </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">ppp pap local-user</font>`<br/><font style="color:rgb(15, 17, 21);"> </font><font style="color:rgb(15, 17, 21);">与对端 AAA 中的用户密码</font> |
| <font style="color:rgb(15, 17, 21);">无法获取 IP</font> | <font style="color:rgb(15, 17, 21);">地址池未配置或</font><font style="color:rgb(15, 17, 21);"> </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">remote address pool</font>`<br/><font style="color:rgb(15, 17, 21);"> </font><font style="color:rgb(15, 17, 21);">未引用</font> | <font style="color:rgb(15, 17, 21);">确认</font><font style="color:rgb(15, 17, 21);"> </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">ip pool ppp</font>`<br/><font style="color:rgb(15, 17, 21);"> </font><font style="color:rgb(15, 17, 21);">存在且网段正确</font> |


**<font style="color:rgb(15, 17, 21);">调试命令</font>**<font style="color:rgb(15, 17, 21);">：</font>

<font style="color:rgb(15, 17, 21);">bash</font>

```plain
debugging ppp all                 # 打开 PPP 调试（慎用）
terminal debugging
terminal monitor
```

---

## <font style="color:rgb(15, 17, 21);">五、配置要点总结表</font>
| <font style="color:rgb(15, 17, 21);">配置项</font> | <font style="color:rgb(15, 17, 21);">AR1</font> | <font style="color:rgb(15, 17, 21);">AR2</font> |
| --- | --- | --- |
| <font style="color:rgb(15, 17, 21);">本端认证模式</font> | `<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">ppp authentication-mode chap</font>` | `<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">ppp authentication-mode pap</font>` |
| <font style="color:rgb(15, 17, 21);">本端作为被认证方凭证</font> | `<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">ppp pap local-user wakin password 123</font>` | `<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">ppp chap user winnie</font>`<br/><font style="color:rgb(15, 17, 21);"> </font><font style="color:rgb(15, 17, 21);">+</font><font style="color:rgb(15, 17, 21);"> </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">ppp chap password 456</font>` |
| <font style="color:rgb(15, 17, 21);">对端认证所需本地用户</font> | `<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">local-user winnie</font>`<br/><font style="color:rgb(15, 17, 21);"> </font><font style="color:rgb(15, 17, 21);">(CHAP)</font> | `<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">local-user wakin</font>`<br/><font style="color:rgb(15, 17, 21);"> </font><font style="color:rgb(15, 17, 21);">(PAP)</font> |
| <font style="color:rgb(15, 17, 21);">地址分配</font> | `<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">remote address pool ppp</font>` | `<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">ip address ppp-negotiate</font>` |
| <font style="color:rgb(15, 17, 21);">缺省路由</font> | <font style="color:rgb(15, 17, 21);">—</font> | `<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">ppp ipcp default-route</font>` |


---

## <font style="color:rgb(15, 17, 21);">六、验证清单</font>
+ <font style="color:rgb(15, 17, 21);">AR1 和 AR2 的</font><font style="color:rgb(15, 17, 21);"> </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">Pos6/0/0</font>`<font style="color:rgb(15, 17, 21);"> </font><font style="color:rgb(15, 17, 21);">协议状态均为</font><font style="color:rgb(15, 17, 21);"> </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">UP</font>`<font style="color:rgb(15, 17, 21);">。</font>
+ <font style="color:rgb(15, 17, 21);">AR2 接口获得</font><font style="color:rgb(15, 17, 21);"> </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">9.1.1.x</font>`<font style="color:rgb(15, 17, 21);"> </font><font style="color:rgb(15, 17, 21);">的 IP（</font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">display interface Pos6/0/0</font>`<font style="color:rgb(15, 17, 21);">）。</font>
+ <font style="color:rgb(15, 17, 21);">AR2 路由表中有缺省路由指向</font><font style="color:rgb(15, 17, 21);"> </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">9.1.1.1</font>`<font style="color:rgb(15, 17, 21);">。</font>
+ <font style="color:rgb(15, 17, 21);">AR1 能 ping 通 AR2 的接口 IP。</font>
+ <font style="color:rgb(15, 17, 21);">抓包可看到 LCP → CHAP → PAP → IPCP 的顺序。</font>
