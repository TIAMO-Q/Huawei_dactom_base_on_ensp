  <img src="https://cdn.nlark.com/yuque/0/2026/png/50508853/1776512558290-a1dba5d8-2c76-4fa6-a71a-d6efa6fdf0e1.png" width="594" title="" crop="0,0,1,1" id="u5254ecb6" class="ne-image">

## <font style="color:rgb(15, 17, 21);">一、拓扑结构与桥 ID 整理</font>
| <font style="color:rgb(15, 17, 21);">设备</font> | <font style="color:rgb(15, 17, 21);">桥 ID（优先级.MAC）</font> | <font style="color:rgb(15, 17, 21);">优先级</font> | <font style="color:rgb(15, 17, 21);">MAC 地址</font> |
| --- | --- | --- | --- |
| **<font style="color:rgb(15, 17, 21);">LSW3</font>** | `<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">0.4c1f-cc33-0351</font>` | <font style="color:rgb(15, 17, 21);">0</font> | <font style="color:rgb(15, 17, 21);">4c1f-cc33-0351</font> |
| **<font style="color:rgb(15, 17, 21);">LSW4</font>** | `<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">4096.4c1f-cc21-66af</font>` | <font style="color:rgb(15, 17, 21);">4096</font> | <font style="color:rgb(15, 17, 21);">4c1f-cc21-66af</font> |
| **<font style="color:rgb(15, 17, 21);">LSW2</font>** | `<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">32768.4c1f-cc16-0af7</font>` | <font style="color:rgb(15, 17, 21);">32768</font> | <font style="color:rgb(15, 17, 21);">4c1f-cc16-0af7</font> |
| **<font style="color:rgb(15, 17, 21);">LSW1</font>** | `<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">32768.4c1f-ccc3-079b</font>` | <font style="color:rgb(15, 17, 21);">32768</font> | <font style="color:rgb(15, 17, 21);">4c1f-ccc3-079b</font> |


<font style="color:rgb(15, 17, 21);">注意：桥 ID 中优先级数值越小越优，优先级相同则比较 MAC 地址，MAC 越小越优。因此根桥是</font><font style="color:rgb(15, 17, 21);"> </font>**<font style="color:rgb(15, 17, 21);">LSW3</font>**<font style="color:rgb(15, 17, 21);">（优先级 0 最小）。</font>

---

## <font style="color:rgb(15, 17, 21);">二、STP 选举结果分析（根据图中角色）</font>
### <font style="color:rgb(15, 17, 21);">1. 根桥选举</font>
+ <font style="color:rgb(15, 17, 21);">全网比较桥 ID，LSW3 优先级为 0（最小），成为</font>**<font style="color:rgb(15, 17, 21);">根桥</font>**<font style="color:rgb(15, 17, 21);">。</font>
+ <font style="color:rgb(15, 17, 21);">根桥的所有端口都是</font>**<font style="color:rgb(15, 17, 21);">指定端口（DP）</font>**<font style="color:rgb(15, 17, 21);">。</font>

### <font style="color:rgb(15, 17, 21);">2. 根端口（RP）选举（非根桥上）</font>
+ **<font style="color:rgb(15, 17, 21);">LSW4</font>**<font style="color:rgb(15, 17, 21);">：连接到 LSW3 的端口（E0/0/1？）应成为根端口。图中 LSW3 的 E0/0/1 连接 LSW4 的某端口，且 LSW4 上该端口角色为</font><font style="color:rgb(15, 17, 21);"> </font>**<font style="color:rgb(15, 17, 21);">RP</font>**<font style="color:rgb(15, 17, 21);">。</font>
+ **<font style="color:rgb(15, 17, 21);">LSW2</font>**<font style="color:rgb(15, 17, 21);">：连接到 LSW4 的端口（G0/0/1）是根端口？图中 LSW2 的 G0/0/1 标记为</font><font style="color:rgb(15, 17, 21);"> </font>**<font style="color:rgb(15, 17, 21);">RP</font>**<font style="color:rgb(15, 17, 21);">，说明该端口到根桥的路径最短。</font>
+ **<font style="color:rgb(15, 17, 21);">LSW1</font>**<font style="color:rgb(15, 17, 21);">：连接到 LSW2 的两个端口（G0/0/1 和 G0/0/2），其中一个是根端口（RP），另一个是替代端口（AP）。图中 LSW1 的 G0/0/1 是</font><font style="color:rgb(15, 17, 21);"> </font>**<font style="color:rgb(15, 17, 21);">RP</font>**<font style="color:rgb(15, 17, 21);">，G0/0/2 是</font><font style="color:rgb(15, 17, 21);"> </font>**<font style="color:rgb(15, 17, 21);">AP</font>**<font style="color:rgb(15, 17, 21);">。</font>

### <font style="color:rgb(15, 17, 21);">3. 指定端口（DP）选举</font>
+ <font style="color:rgb(15, 17, 21);">每条链路上，到达根桥路径开销最小的端口成为 DP。</font>
+ <font style="color:rgb(15, 17, 21);">根桥所有端口都是 DP。</font>
+ <font style="color:rgb(15, 17, 21);">非根桥上，连接到下游交换机的端口如果在该网段上路径开销更优，则成为 DP。</font>
    - <font style="color:rgb(15, 17, 21);">例如 LSW4 上连接 LSW2 的端口（E0/0/2？）被标记为</font><font style="color:rgb(15, 17, 21);"> </font>**<font style="color:rgb(15, 17, 21);">DP</font>**<font style="color:rgb(15, 17, 21);">。</font>
    - <font style="color:rgb(15, 17, 21);">LSW2 上连接 LSW1 的两个端口：其中一个是 DP（G0/0/3？），另一个不是。</font>

### <font style="color:rgb(15, 17, 21);">4. 阻塞端口（AP）</font>
+ <font style="color:rgb(15, 17, 21);">既不是 RP 也不是 DP 的端口成为 AP（Alternate Port），逻辑阻塞。</font>
+ <font style="color:rgb(15, 17, 21);">图中 LSW1 的 G0/0/2 是 AP。</font>

---

## <font style="color:rgb(15, 17, 21);">三、STP 知识点详解</font>
### <font style="color:rgb(15, 17, 21);">1. 桥 ID（BID）</font>
+ <font style="color:rgb(15, 17, 21);">格式：</font>**<font style="color:rgb(15, 17, 21);">优先级（2 字节） + MAC 地址（6 字节）</font>**
+ <font style="color:rgb(15, 17, 21);">优先级取值范围 0~61440，步长为 4096（默认 32768）。</font>
+ <font style="color:rgb(15, 17, 21);">比较规则：先比优先级，数值越小越优；优先级相同比 MAC 地址，数值越小越优。</font>

### <font style="color:rgb(15, 17, 21);">2. 路径开销（Path Cost）</font>
+ <font style="color:rgb(15, 17, 21);">与链路带宽有关，带宽越高开销越小。华为设备默认开销标准为</font><font style="color:rgb(15, 17, 21);"> </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">dot1t</font>`<font style="color:rgb(15, 17, 21);">。</font>
+ <font style="color:rgb(15, 17, 21);">常见开销（100 Mbps 参考带宽）：</font>
    - <font style="color:rgb(15, 17, 21);">10 Mbps → 2,000,000</font>
    - <font style="color:rgb(15, 17, 21);">100 Mbps → 200,000</font>
    - <font style="color:rgb(15, 17, 21);">1000 Mbps → 20,000</font>
    - <font style="color:rgb(15, 17, 21);">10000 Mbps → 2,000</font>

### <font style="color:rgb(15, 17, 21);">3. 端口 ID（PID）</font>
+ <font style="color:rgb(15, 17, 21);">格式：</font>**<font style="color:rgb(15, 17, 21);">端口优先级（1 字节） + 端口号（1 字节）</font>**
+ <font style="color:rgb(15, 17, 21);">端口优先级默认 128，范围 0~240，步长 16。</font>
+ <font style="color:rgb(15, 17, 21);">在根端口选举中，当路径开销和对端桥 ID 都相同时，比较对端端口 ID（PID 越小越优）。</font>

### <font style="color:rgb(15, 17, 21);">4. 端口角色</font>
+ **<font style="color:rgb(15, 17, 21);">根端口（Root Port, RP）</font>**<font style="color:rgb(15, 17, 21);">：非根桥上到达根桥的最优端口（路径开销最小）。</font>
+ **<font style="color:rgb(15, 17, 21);">指定端口（Designated Port, DP）</font>**<font style="color:rgb(15, 17, 21);">：每个网段上到达根桥路径开销最小的端口（根桥上所有端口都是 DP）。</font>
+ **<font style="color:rgb(15, 17, 21);">替代端口（Alternate Port, AP）</font>**<font style="color:rgb(15, 17, 21);">：被阻塞的端口，是根端口的备份。</font>
+ **<font style="color:rgb(15, 17, 21);">备份端口（Backup Port, BP）</font>**<font style="color:rgb(15, 17, 21);">：同一台交换机的两个端口之间形成环路时的阻塞端口（极少见）。</font>

### <font style="color:rgb(15, 17, 21);">5. 端口状态</font>
| <font style="color:rgb(15, 17, 21);">状态</font> | <font style="color:rgb(15, 17, 21);">说明</font> |
| --- | --- |
| <font style="color:rgb(15, 17, 21);">Disabled</font> | <font style="color:rgb(15, 17, 21);">端口 down 或被 shutdown</font> |
| <font style="color:rgb(15, 17, 21);">Blocking</font> | <font style="color:rgb(15, 17, 21);">仅接收 BPDU，不转发数据（AP 状态）</font> |
| <font style="color:rgb(15, 17, 21);">Listening</font> | <font style="color:rgb(15, 17, 21);">选举阶段，收发 BPDU，不转发数据</font> |
| <font style="color:rgb(15, 17, 21);">Learning</font> | <font style="color:rgb(15, 17, 21);">学习 MAC 地址，不转发数据</font> |
| <font style="color:rgb(15, 17, 21);">Forwarding</font> | <font style="color:rgb(15, 17, 21);">正常转发数据</font> |


**<font style="color:rgb(15, 17, 21);">BPDU 就是 STP 的“语言”，交换机通过它来协商生成树，避免环路，并在网络变化时快速调整</font>**

### <font style="color:rgb(15, 17, 21);">6. 选举过程</font>
1. <font style="color:rgb(15, 17, 21);">选根桥（最小 BID）</font>
2. <font style="color:rgb(15, 17, 21);">每台非根桥选根端口（最小路径开销 → 最小对端 BID → 最小对端 PID）</font>
3. <font style="color:rgb(15, 17, 21);">每个网段选指定端口（最小路径开销 → 最小本端 BID → 最小本端 PID）</font>
4. <font style="color:rgb(15, 17, 21);">剩余端口阻塞</font>

---

## <font style="color:rgb(15, 17, 21);">四、配置命令（华为设备）</font>
### <font style="color:rgb(15, 17, 21);">1. 设置根桥和备份根桥</font>
<font style="color:rgb(15, 17, 21);">bash</font>

```plain
# 将 LSW3 设为根桥（优先级 0）
[LSW3] stp root primary
# 或者手动设置优先级为 0（步长 4096）
[LSW3] stp priority 0

# 将 LSW4 设为备份根桥（优先级 4096）
[LSW4] stp root secondary
# 或手动设置优先级 4096
[LSW4] stp priority 4096
```

### <font style="color:rgb(15, 17, 21);">2. 修改端口优先级（影响指定端口选举）</font>
<font style="color:rgb(15, 17, 21);">例如，为了让 LSW2 的某个端口成为指定端口而非阻塞，可调高其优先级（数值越小越优）：</font>

<font style="color:rgb(15, 17, 21);">bash</font>

```plain
[LSW2] interface GigabitEthernet0/0/3
[LSW2-GigabitEthernet0/0/3] stp priority 16   # 降低数值，提高优先级
```

### <font style="color:rgb(15, 17, 21);">3. 修改端口路径开销（影响根端口选举）</font>
<font style="color:rgb(15, 17, 21);">bash</font>

```plain
[LSW2] interface GigabitEthernet0/0/1
[LSW2-GigabitEthernet0/0/1] stp cost 20000   # 手动指定开销
```

### <font style="color:rgb(15, 17, 21);">4. 查看 STP 信息</font>
<font style="color:rgb(15, 17, 21);">bash</font>

```plain
display stp                              # 查看全局 STP 状态
display stp brief                        # 查看端口角色和状态
display stp interface GigabitEthernet0/0/1  # 查看指定接口的 STP 信息
display stp root                         # 查看根桥信息
```

### <font style="color:rgb(15, 17, 21);">5. 配置边缘端口（加速收敛，用于连接终端）</font>
<font style="color:rgb(15, 17, 21);">bash</font>

```plain
[LSW1] interface Ethernet0/0/1
[LSW1-Ethernet0/0/1] stp edged-port enable
```

### <font style="color:rgb(15, 17, 21);">6. 启用 BPDU 保护（防止非法 BPDU 攻击）</font>
<font style="color:rgb(15, 17, 21);">bash</font>

<font style="color:rgb(15, 17, 21);">[LSW1] stp bpdu-protection</font>

---

## <font style="color:rgb(15, 17, 21);">五、验证你图中的角色是否正确</font>
<font style="color:rgb(15, 17, 21);">根据你提供的桥 ID 和连接关系，我们可以手动计算验证：</font>

+ **<font style="color:rgb(15, 17, 21);">根桥</font>**<font style="color:rgb(15, 17, 21);">：LSW3（优先级 0），所有端口为 DP。</font>
+ **<font style="color:rgb(15, 17, 21);">LSW4 到根桥</font>**<font style="color:rgb(15, 17, 21);">：只有一条链路（E0/0/1 → LSW3 E0/0/1），该端口成为 RP。</font>
+ **<font style="color:rgb(15, 17, 21);">LSW4 到 LSW2</font>**<font style="color:rgb(15, 17, 21);">：假设 LSW4 的 E0/0/2 连接 LSW2 的 G0/0/1。LSW4 到根桥开销小于 LSW2 到根桥的开销（因为 LSW4 离根桥更近），因此 LSW4 的 E0/0/2 成为 DP。</font>
+ **<font style="color:rgb(15, 17, 21);">LSW2</font>**<font style="color:rgb(15, 17, 21);">：其 G0/0/1 收到 LSW4 的 BPDU，由于 LSW4 的 DP 宣告了更优的路径，LSW2 的 G0/0/1 成为 RP。</font>
+ **<font style="color:rgb(15, 17, 21);">LSW2 到 LSW1</font>**<font style="color:rgb(15, 17, 21);">：两条链路（G0/0/3 和 G0/0/4）。LSW2 到根桥的路径开销相同（假设两条链路带宽一样），则比较对端桥 ID（LSW1 的 BID 相同），再比较对端端口 ID，端口号小的（G0/0/1？）成为 RP，另一个成为 AP。图中 LSW1 的 G0/0/1 是 RP，G0/0/2 是 AP，符合预期。</font>

---

## <font style="color:rgb(15, 17, 21);">六、总结</font>
+ <font style="color:rgb(15, 17, 21);">STP 通过选举根桥、根端口、指定端口，逻辑阻断冗余链路，防止环路。</font>
+ <font style="color:rgb(15, 17, 21);">桥 ID 决定根桥，路径开销和端口 ID 决定根端口和指定端口。</font>
+ <font style="color:rgb(15, 17, 21);">华为设备默认启用 MSTP（多实例生成树），但单实例行为与 STP/RSTP 类似。</font>
+ <font style="color:rgb(15, 17, 21);">配置命令主要集中在 </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">stp priority</font>`<font style="color:rgb(15, 17, 21);">、</font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">stp cost</font>`<font style="color:rgb(15, 17, 21);">、</font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">stp port priority</font>`<font style="color:rgb(15, 17, 21);"> 和 </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">stp edged-port</font>`<font style="color:rgb(15, 17, 21);"> 等。</font>
