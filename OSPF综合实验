<img src="https://cdn.nlark.com/yuque/0/2026/png/50508853/1776656187891-bd6ebf57-55ee-4015-a35b-07ca0306274a.png" width="936.6666666666666" title="" crop="0,0,1,1" id="u7ff525af" class="ne-image">

### <font style="color:rgb(15, 17, 21);">拓扑结构与角色推断</font>
+ **<font style="color:rgb(15, 17, 21);">广播网络</font>**<font style="color:rgb(15, 17, 21);">：R1、R2、R3、R6 通过交换机（图中未画出）连接在</font><font style="color:rgb(15, 17, 21);"> </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">192.168.1.0/24</font>`<font style="color:rgb(15, 17, 21);"> </font><font style="color:rgb(15, 17, 21);">网段，运行 OSPF Area 0。</font>
+ **<font style="color:rgb(15, 17, 21);">点对点链路</font>**<font style="color:rgb(15, 17, 21);">：R3 与 R4 通过串口（</font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">S2/0/0</font>`<font style="color:rgb(15, 17, 21);"> </font><font style="color:rgb(15, 17, 21);">–</font><font style="color:rgb(15, 17, 21);"> </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">S4/0/0</font>`<font style="color:rgb(15, 17, 21);">）连接，网段</font><font style="color:rgb(15, 17, 21);"> </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">34.0.0.0/24</font>`<font style="color:rgb(15, 17, 21);">，也属于 Area 0，并配置 MD5 认证。</font>
+ **<font style="color:rgb(15, 17, 21);">外部网络</font>**<font style="color:rgb(15, 17, 21);">：R4 通过</font><font style="color:rgb(15, 17, 21);"> </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">GE0/0/0</font>`<font style="color:rgb(15, 17, 21);"> </font><font style="color:rgb(15, 17, 21);">连接</font><font style="color:rgb(15, 17, 21);"> </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">48.0.0.0/24</font>`<font style="color:rgb(15, 17, 21);"> </font><font style="color:rgb(15, 17, 21);">网段，与 R8 互联。R8 模拟 ISP，有 Loopback</font><font style="color:rgb(15, 17, 21);"> </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">8.8.8.8/32</font>`<font style="color:rgb(15, 17, 21);">。</font>
+ **<font style="color:rgb(15, 17, 21);">ASBR</font>**<font style="color:rgb(15, 17, 21);">：R4 同时作为</font><font style="color:rgb(15, 17, 21);"> </font>**<font style="color:rgb(15, 17, 21);">ASBR</font>**<font style="color:rgb(15, 17, 21);">（自治系统边界路由器），通过静态默认路由指向 R8，并使用</font><font style="color:rgb(15, 17, 21);"> </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">default-route-advertise</font>`<font style="color:rgb(15, 17, 21);"> </font><font style="color:rgb(15, 17, 21);">向 OSPF 域内注入默认路由。</font>

---

## <font style="color:rgb(15, 17, 21);">一、OSPF DR/BDR 选举（广播网络）</font>
### <font style="color:rgb(15, 17, 21);">1.1 优先级配置与选举结果</font>
| <font style="color:rgb(15, 17, 21);">路由器</font> | <font style="color:rgb(15, 17, 21);">接口</font> | <font style="color:rgb(15, 17, 21);">优先级</font> | <font style="color:rgb(15, 17, 21);">角色</font> |
| --- | --- | --- | --- |
| <font style="color:rgb(15, 17, 21);">R1</font> | <font style="color:rgb(15, 17, 21);">GE0/0/0</font> | <font style="color:rgb(15, 17, 21);">200</font> | **<font style="color:rgb(15, 17, 21);">DR</font>**<font style="color:rgb(15, 17, 21);">（指定路由器）</font> |
| <font style="color:rgb(15, 17, 21);">R2</font> | <font style="color:rgb(15, 17, 21);">GE0/0/0</font> | <font style="color:rgb(15, 17, 21);">100</font> | **<font style="color:rgb(15, 17, 21);">BDR</font>**<font style="color:rgb(15, 17, 21);">（备份指定路由器）</font> |
| <font style="color:rgb(15, 17, 21);">R3</font> | <font style="color:rgb(15, 17, 21);">GE0/0/0</font> | <font style="color:rgb(15, 17, 21);">0</font> | **<font style="color:rgb(15, 17, 21);">DROther</font>**<font style="color:rgb(15, 17, 21);">（不参与选举）</font> |
| <font style="color:rgb(15, 17, 21);">R6</font> | <font style="color:rgb(15, 17, 21);">GE0/0/0</font> | <font style="color:rgb(15, 17, 21);">0</font> | **<font style="color:rgb(15, 17, 21);">DROther</font>** |


**<font style="color:rgb(15, 17, 21);">配置命令</font>**<font style="color:rgb(15, 17, 21);">（R1 示例）：</font>

<font style="color:rgb(15, 17, 21);">bash</font>

```plain
[R1] interface GigabitEthernet0/0/0
[R1-GigabitEthernet0/0/0] ip address 192.168.1.1 24
[R1-GigabitEthernet0/0/0] ospf dr-priority 200
[R1] ospf 1 router-id 1.1.1.1
[R1-ospf-1] area 0
[R1-ospf-1-area-0.0.0.0] network 192.168.1.1 0.0.0.0
```

**<font style="color:rgb(15, 17, 21);">关键点</font>**<font style="color:rgb(15, 17, 21);">：</font>

+ `<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">ospf dr-priority</font>`<font style="color:rgb(15, 17, 21);"> </font><font style="color:rgb(15, 17, 21);">取值范围 0~255，默认 1。优先级</font><font style="color:rgb(15, 17, 21);"> </font>**<font style="color:rgb(15, 17, 21);">0</font>**<font style="color:rgb(15, 17, 21);"> </font><font style="color:rgb(15, 17, 21);">表示不参与选举，只能成为 DROther。</font>
+ <font style="color:rgb(15, 17, 21);">DR/BDR 选举</font>**<font style="color:rgb(15, 17, 21);">非抢占</font>**<font style="color:rgb(15, 17, 21);">：即使后来加入更高优先级的设备，也不会替换现有 DR/BDR，除非它们失效。</font>
+ <font style="color:rgb(15, 17, 21);">查看命令：</font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">display ospf interface GigabitEthernet0/0/0</font>`<font style="color:rgb(15, 17, 21);"> </font><font style="color:rgb(15, 17, 21);">可看到当前 DR/BDR 的 Router ID。</font>

### <font style="color:rgb(15, 17, 21);">1.2 为什么串口不需要选举？</font>
<font style="color:rgb(15, 17, 21);">串口（</font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">S2/0/0</font>`<font style="color:rgb(15, 17, 21);">）的 OSPF 网络类型默认为</font><font style="color:rgb(15, 17, 21);"> </font>**<font style="color:rgb(15, 17, 21);">P2P（点到点）</font>**<font style="color:rgb(15, 17, 21);">，不进行 DR/BDR 选举，直接与对端建立 Full 邻接关系。</font>

---

## <font style="color:rgb(15, 17, 21);">二、OSPF MD5 认证（接口级）</font>
### <font style="color:rgb(15, 17, 21);">2.1 配置解析（R3 与 R4 之间）</font>
<font style="color:rgb(15, 17, 21);">bash</font>

```plain
[R3] interface Serial2/0/0
[R3-Serial2/0/0] ip address 34.0.0.3 24
[R3-Serial2/0/0] ospf authentication-mode md5 1 cipher 123.com
```

**<font style="color:rgb(15, 17, 21);">命令拆解</font>**<font style="color:rgb(15, 17, 21);">：</font>

+ `<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">authentication-mode md5</font>`<font style="color:rgb(15, 17, 21);">：使用 MD5 算法进行认证（非明文）。</font>
+ `<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">1</font>`<font style="color:rgb(15, 17, 21);">：</font>**<font style="color:rgb(15, 17, 21);">密钥 ID</font>**<font style="color:rgb(15, 17, 21);">（Key ID），范围 1~255。两端必须一致，用于支持密钥滚动。</font>
+ `<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">cipher</font>`<font style="color:rgb(15, 17, 21);">：表示密码以密文形式存储在配置中（</font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">cipher 123.com</font>`<font style="color:rgb(15, 17, 21);"> </font><font style="color:rgb(15, 17, 21);">实际存储为加密字符串）。</font>
+ <font style="color:rgb(15, 17, 21);">密码</font><font style="color:rgb(15, 17, 21);"> </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">123.com</font>`<font style="color:rgb(15, 17, 21);">：两端必须相同。</font>

<font style="color:rgb(15, 17, 21);">R4 配置类似：</font>

<font style="color:rgb(15, 17, 21);">bash</font>

```plain
[R4] interface Serial4/0/0
[R4-Serial4/0/0] ip address 34.0.0.4 24
[R4-Serial4/0/0] ospf authentication-mode md5 1 cipher 123.com
```

**<font style="color:rgb(15, 17, 21);">抓包验证</font>**<font style="color:rgb(15, 17, 21);">：</font>

+ <font style="color:rgb(15, 17, 21);">过滤</font><font style="color:rgb(15, 17, 21);"> </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">ospf</font>`<font style="color:rgb(15, 17, 21);">，查看 Hello 报文尾部会增加</font><font style="color:rgb(15, 17, 21);"> </font>**<font style="color:rgb(15, 17, 21);">MD5 认证字段</font>**<font style="color:rgb(15, 17, 21);">（包含 Key ID、MD5 摘要等）。</font>
+ <font style="color:rgb(15, 17, 21);">如果密码错误，邻居状态会卡在</font><font style="color:rgb(15, 17, 21);"> </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">ExStart</font>`<font style="color:rgb(15, 17, 21);"> </font><font style="color:rgb(15, 17, 21);">或</font><font style="color:rgb(15, 17, 21);"> </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">Exchange</font>`<font style="color:rgb(15, 17, 21);">，并产生</font><font style="color:rgb(15, 17, 21);"> </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">Authentication failure</font>`<font style="color:rgb(15, 17, 21);"> </font><font style="color:rgb(15, 17, 21);">日志。</font>

---

## <font style="color:rgb(15, 17, 21);">三、默认路由注入（ASBR 行为）</font>
### <font style="color:rgb(15, 17, 21);">3.1 R4 作为 ASBR 的配置</font>
<font style="color:rgb(15, 17, 21);">bash</font>

```plain
[R4] ip route-static 0.0.0.0 0 48.0.0.8   # 指向 ISP 的静态默认路由
[R4] ospf 1 router-id 4.4.4.4
[R4-ospf-1] default-route-advertise        # 向 OSPF 域内发布默认路由
[R4-ospf-1] area 0
[R4-ospf-1-area-0.0.0.0] network 34.0.0.4 0.0.0.0
[R4-ospf-1-area-0.0.0.0] network 4.4.4.4 0.0.0.0
```

**<font style="color:rgb(15, 17, 21);">命令解析</font>**<font style="color:rgb(15, 17, 21);">：</font>

+ `<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">default-route-advertise</font>`<font style="color:rgb(15, 17, 21);">：ASBR 向 OSPF 域内发布一条</font><font style="color:rgb(15, 17, 21);"> </font>**<font style="color:rgb(15, 17, 21);">Type 5 LSA</font>**<font style="color:rgb(15, 17, 21);">（AS External LSA），通告默认路由</font><font style="color:rgb(15, 17, 21);"> </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">0.0.0.0/0</font>`<font style="color:rgb(15, 17, 21);">。</font>
    - **<font style="color:rgb(15, 17, 21);">前提</font>**<font style="color:rgb(15, 17, 21);">：ASBR 自身必须有默认路由（此处通过静态路由满足）。</font>
    - <font style="color:rgb(15, 17, 21);">可选参数</font><font style="color:rgb(15, 17, 21);"> </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">always</font>`<font style="color:rgb(15, 17, 21);">：即使本地没有默认路由也强制发布（本实验不需要）。</font>
+ <font style="color:rgb(15, 17, 21);">Type 5 LSA 的</font><font style="color:rgb(15, 17, 21);"> </font>**<font style="color:rgb(15, 17, 21);">Link State ID</font>**<font style="color:rgb(15, 17, 21);"> </font><font style="color:rgb(15, 17, 21);">为</font><font style="color:rgb(15, 17, 21);"> </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">0.0.0.0</font>`<font style="color:rgb(15, 17, 21);">，掩码</font><font style="color:rgb(15, 17, 21);"> </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">0.0.0.0</font>`<font style="color:rgb(15, 17, 21);">，Metric 类型通常为 Type 2（外部开销不累加）。</font>

**<font style="color:rgb(15, 17, 21);">其他路由器（R1、R2、R3、R6）学习效果</font>**<font style="color:rgb(15, 17, 21);">：</font>

+ <font style="color:rgb(15, 17, 21);">通过 OSPF 学习到一条缺省路由，下一跳指向 R4（具体为到达 R4 的路径）。</font>
+ <font style="color:rgb(15, 17, 21);">路由表显示：</font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">0.0.0.0/0 [OSPF 150] via 下一跳</font>`<font style="color:rgb(15, 17, 21);">（OSPF 外部路由优先级 150）。</font>

### <font style="color:rgb(15, 17, 21);">3.2 外部路由器 R8 配置</font>
<font style="color:rgb(15, 17, 21);">bash</font>

```plain
[R8] interface GigabitEthernet0/0/0
[R8-GigabitEthernet0/0/0] ip address 48.0.0.8 24
[R8] interface LoopBack8
[R8-LoopBack8] ip address 8.8.8.8 32
[R8] ip route-static 0.0.0.0 0 48.0.0.4   # 回程指向 R4
```

**<font style="color:rgb(15, 17, 21);">说明</font>**<font style="color:rgb(15, 17, 21);">：R8 配置静态默认路由指向 R4，保证外网访问内网的回程可达。</font>

---

## <font style="color:rgb(15, 17, 21);">四、OSPF 邻居建立过程（关键状态机）</font>
| <font style="color:rgb(15, 17, 21);">状态</font> | <font style="color:rgb(15, 17, 21);">发生事件</font> | <font style="color:rgb(15, 17, 21);">报文</font> |
| --- | --- | --- |
| **<font style="color:rgb(15, 17, 21);">Down</font>** | <font style="color:rgb(15, 17, 21);">未收到 Hello</font> | <font style="color:rgb(15, 17, 21);">—</font> |
| **<font style="color:rgb(15, 17, 21);">Init</font>** | <font style="color:rgb(15, 17, 21);">收到 Hello，但报文中无自己 Router ID</font> | <font style="color:rgb(15, 17, 21);">Hello</font> |
| **<font style="color:rgb(15, 17, 21);">2-Way</font>** | <font style="color:rgb(15, 17, 21);">收到 Hello，且报文中包含自己 Router ID（邻居关系建立）</font> | <font style="color:rgb(15, 17, 21);">Hello</font> |
| **<font style="color:rgb(15, 17, 21);">ExStart</font>** | <font style="color:rgb(15, 17, 21);">开始交换 DD 报文，协商主从（Router ID 大者为主）</font> | <font style="color:rgb(15, 17, 21);">DD（空）</font> |
| **<font style="color:rgb(15, 17, 21);">Exchange</font>** | <font style="color:rgb(15, 17, 21);">交换 DD 报文描述 LSDB 摘要</font> | <font style="color:rgb(15, 17, 21);">DD（含 LSA 头）</font> |
| **<font style="color:rgb(15, 17, 21);">Loading</font>** | <font style="color:rgb(15, 17, 21);">通过 LSR 请求缺失 LSA，通过 LSU 发送完整 LSA</font> | <font style="color:rgb(15, 17, 21);">LSR, LSU, LSAck</font> |
| **<font style="color:rgb(15, 17, 21);">Full</font>** | <font style="color:rgb(15, 17, 21);">LSDB 同步完成，邻接关系建立</font> | <font style="color:rgb(15, 17, 21);">—</font> |


**<font style="color:rgb(15, 17, 21);">抓包验证</font>**<font style="color:rgb(15, 17, 21);">：</font>

+ <font style="color:rgb(15, 17, 21);">在 R3 与 R4 的串口上抓包，过滤</font><font style="color:rgb(15, 17, 21);"> </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">ospf</font>`<font style="color:rgb(15, 17, 21);">，可观察到：</font>
    1. <font style="color:rgb(15, 17, 21);">Hello（每 10 秒）</font>
    2. <font style="color:rgb(15, 17, 21);">DD 报文（协商 MTU、主从）</font>
    3. <font style="color:rgb(15, 17, 21);">LSR/LSU 交换（LSA 同步）</font>
    4. <font style="color:rgb(15, 17, 21);">邻居状态变为 Full 后，仅周期性 Hello 维持。</font>

---

## <font style="color:rgb(15, 17, 21);">五、LSA 类型与作用</font>
| <font style="color:rgb(15, 17, 21);">类型</font> | <font style="color:rgb(15, 17, 21);">名称</font> | <font style="color:rgb(15, 17, 21);">产生者</font> | <font style="color:rgb(15, 17, 21);">传播范围</font> | <font style="color:rgb(15, 17, 21);">本实验中实例</font> |
| --- | --- | --- | --- | --- |
| <font style="color:rgb(15, 17, 21);">Type 1</font> | <font style="color:rgb(15, 17, 21);">Router LSA</font> | <font style="color:rgb(15, 17, 21);">每台路由器</font> | <font style="color:rgb(15, 17, 21);">本区域</font> | <font style="color:rgb(15, 17, 21);">R1 描述自身接口</font> |
| <font style="color:rgb(15, 17, 21);">Type 2</font> | <font style="color:rgb(15, 17, 21);">Network LSA</font> | <font style="color:rgb(15, 17, 21);">DR</font> | <font style="color:rgb(15, 17, 21);">本区域</font> | <font style="color:rgb(15, 17, 21);">由 DR (R1) 描述广播网络</font><font style="color:rgb(15, 17, 21);"> </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">192.168.1.0/24</font>`<br/><font style="color:rgb(15, 17, 21);"> </font><font style="color:rgb(15, 17, 21);">中的所有路由器</font> |
| <font style="color:rgb(15, 17, 21);">Type 3</font> | <font style="color:rgb(15, 17, 21);">Network Summary LSA</font> | <font style="color:rgb(15, 17, 21);">ABR</font> | <font style="color:rgb(15, 17, 21);">其他区域</font> | <font style="color:rgb(15, 17, 21);">本实验无 ABR（都在 Area 0），无此 LSA</font> |
| <font style="color:rgb(15, 17, 21);">Type 4</font> | <font style="color:rgb(15, 17, 21);">ASBR Summary LSA</font> | <font style="color:rgb(15, 17, 21);">ABR</font> | <font style="color:rgb(15, 17, 21);">其他区域</font> | <font style="color:rgb(15, 17, 21);">无</font> |
| <font style="color:rgb(15, 17, 21);">Type 5</font> | <font style="color:rgb(15, 17, 21);">AS External LSA</font> | <font style="color:rgb(15, 17, 21);">ASBR</font> | <font style="color:rgb(15, 17, 21);">整个 AS</font> | <font style="color:rgb(15, 17, 21);">R4 发布的默认路由</font><font style="color:rgb(15, 17, 21);"> </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">0.0.0.0/0</font>` |


**<font style="color:rgb(15, 17, 21);">查看命令</font>**<font style="color:rgb(15, 17, 21);">：</font>

<font style="color:rgb(15, 17, 21);">bash</font>

```plain
display ospf lsdb               # 查看 LSDB 摘要
display ospf lsdb router 1.1.1.1   # 查看 R1 的 Router LSA
display ospf lsdb ase 0.0.0.0      # 查看默认路由的外部 LSA
```

---

## <font style="color:rgb(15, 17, 21);">六、验证清单</font>
| <font style="color:rgb(15, 17, 21);">验证项</font> | <font style="color:rgb(15, 17, 21);">命令</font> | <font style="color:rgb(15, 17, 21);">预期结果</font> |
| --- | --- | --- |
| <font style="color:rgb(15, 17, 21);">DR/BDR 状态</font> | `<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">display ospf interface GigabitEthernet0/0/0</font>` | <font style="color:rgb(15, 17, 21);">DR: 192.168.1.1 (R1), BDR: 192.168.1.2 (R2)</font> |
| <font style="color:rgb(15, 17, 21);">OSPF 邻居</font> | `<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">display ospf peer</font>` | <font style="color:rgb(15, 17, 21);">所有邻居状态为</font><font style="color:rgb(15, 17, 21);"> </font>**<font style="color:rgb(15, 17, 21);">Full</font>** |
| <font style="color:rgb(15, 17, 21);">MD5 认证</font> | `<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">display ospf error</font>` | <font style="color:rgb(15, 17, 21);">无认证错误计数</font> |
| <font style="color:rgb(15, 17, 21);">默认路由学习</font> | `<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">display ip routing-table 0.0.0.0</font>` | <font style="color:rgb(15, 17, 21);">所有内部路由器（R1,R2,R3,R6）有 OSPF 外部默认路由</font> |
| <font style="color:rgb(15, 17, 21);">连通性测试</font> | `<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">ping 8.8.8.8</font>`<br/><font style="color:rgb(15, 17, 21);">（从 R1）</font> | <font style="color:rgb(15, 17, 21);">通（通过 R4 转发）</font> |
| <font style="color:rgb(15, 17, 21);">抓包验证认证</font> | <font style="color:rgb(15, 17, 21);">过滤</font><font style="color:rgb(15, 17, 21);"> </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">ospf</font>`<br/><font style="color:rgb(15, 17, 21);"> </font><font style="color:rgb(15, 17, 21);">查看 Hello 报文</font> | <font style="color:rgb(15, 17, 21);">尾部有 MD5 摘要字段</font> |


---

## <font style="color:rgb(15, 17, 21);">七、常见排错</font>
| <font style="color:rgb(15, 17, 21);">现象</font> | <font style="color:rgb(15, 17, 21);">可能原因</font> | <font style="color:rgb(15, 17, 21);">解决方法</font> |
| --- | --- | --- |
| <font style="color:rgb(15, 17, 21);">OSPF 邻居卡在 ExStart</font> | <font style="color:rgb(15, 17, 21);">MTU 不一致或认证失败</font> | <font style="color:rgb(15, 17, 21);">检查接口 MTU（</font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">display interface</font>`<br/><font style="color:rgb(15, 17, 21);">），确保 MD5 密码相同</font> |
| <font style="color:rgb(15, 17, 21);">邻居卡在 2-Way 不进入 Full</font> | <font style="color:rgb(15, 17, 21);">广播网络中 DROther 之间正常（仅与 DR/BDR 建立 Full）</font> | <font style="color:rgb(15, 17, 21);">检查 DR/BDR 是否存在；若优先级 0 则正常</font> |
| <font style="color:rgb(15, 17, 21);">默认路由未注入</font> | <font style="color:rgb(15, 17, 21);">ASBR 缺少</font><font style="color:rgb(15, 17, 21);"> </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">default-route-advertise</font>`<br/><font style="color:rgb(15, 17, 21);"> </font><font style="color:rgb(15, 17, 21);">或本地无默认路由</font> | <font style="color:rgb(15, 17, 21);">确认 R4 有静态默认路由且已配置该命令</font> |
| <font style="color:rgb(15, 17, 21);">无法 ping 通 8.8.8.8</font> | <font style="color:rgb(15, 17, 21);">回程路由缺失或 NAT 未配置</font> | <font style="color:rgb(15, 17, 21);">检查 R8 是否有默认路由指向 R4，R4 是否启用 NAT（本实验未涉及 NAT）</font> |


---

## <font style="color:rgb(15, 17, 21);">八、知识点总结表</font>
| <font style="color:rgb(15, 17, 21);">知识点</font> | <font style="color:rgb(15, 17, 21);">关键点</font> |
| --- | --- |
| <font style="color:rgb(15, 17, 21);">DR/BDR 选举</font> | <font style="color:rgb(15, 17, 21);">优先级越高越优，相同则 Router ID 大者优；非抢占；优先级 0 不参与</font> |
| <font style="color:rgb(15, 17, 21);">OSPF 认证（MD5）</font> | <font style="color:rgb(15, 17, 21);">两端 Key ID 和密码必须一致；报文携带 MD5 摘要</font> |
| <font style="color:rgb(15, 17, 21);">默认路由注入</font> | `<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">default-route-advertise</font>`<br/><font style="color:rgb(15, 17, 21);"> </font><font style="color:rgb(15, 17, 21);">产生 Type 5 LSA，前提是 ASBR 自身有默认路由</font> |
| <font style="color:rgb(15, 17, 21);">串口网络类型</font> | <font style="color:rgb(15, 17, 21);">默认为 P2P，无 DR/BDR 选举</font> |
| <font style="color:rgb(15, 17, 21);">LSA 类型</font> | <font style="color:rgb(15, 17, 21);">Type1（Router）、Type2（Network）、Type5（External）</font> |

