### 排错 TS2.0

16 ---> 10

![image-20211016204427388](https://i.loli.net/2021/10/16/pE28Grd67lNyRWC.png)

TS 完整需求16题

[TOC]



#### 1、Eth-Trunk

 Site1中，LSW1-LSW2之间的所有链路要求做eth-trunk的捆绑，并且此eth-trunk要求做 `src-dst-ip` 负载；

##### 可能故障原因：

1. 模式错误，采用 lacp-static
2. 负载方式错误
3. 成员接口未加入或者添加错误
4. Eth-Trunk 未放行所有 VLAN

##### 解决方案：

1. 查看 eth 编号是啥 disp port vlan active
2. 进入 eth12 查看 mode 和 loadbalance
3. 修改 mode 时候报错

```sql
[LSW2-Eth-Trunk12]mode lacp-static 
Error: Error in changing trunk working mode. There is(are) port(s) in the trunk.
```

4. 看到报错提示是 说有端口再用
5. disp eth12 查看是哪个接口再用

   SW1 

```sql
interface Ethernet0/0/18
	port hybrid tagged vlan 1
	# 需要用
	port hybrid untagged vlan 1
```

6. 进入接口，undo eth

7. 进入 eth12，更改模式

##### 验证

SW1、SW2 配置

```sql
interface Eth-Trunk12
	port link-type trunk
	port trunk allow-pass vlan 2 to 4094
	mode lacp-static
	gvrp
```

![image-20211016211539240](https://i.loli.net/2021/10/16/uiSYWL2FJIHCnXa.png)

eth 现象 --- 看到三个成员接口以及负载方式 `According to SIP-XOR-DIP  `, 工作方式 `WorkingMode: STATIC`

```sql
[LSW1]disp eth 12
Local:
LAG ID: 12                  WorkingMode: STATIC                               
Preempt Delay: Disabled     Hash arithmetic: According to SIP-XOR-DIP                         
--------------------------------------------------------------------------------
ActorPortName          Status   PortType PortPri PortNo PortKey PortState Weight
Ethernet0/0/19         Selected 100M     32768   20     3105    10111100  1     
Ethernet0/0/20         Selected 100M     32768   21     3105    10111100  1     
Ethernet0/0/18         Selected 100M     32768   19     3105    10111100  1  
```



#### 2、MSTP

 Site1中，CLIENT1、CLIENT2属于VLAN12，CLIENT3、CLIENT4属于VLAN34；
 MSTP中的VLAN12属于instance1，vlan34属于instance2；两个instance的主备根桥分别在SW1和SW2上，
 要求CLIENT1访问R1时经过的路径是SW3-SW1-R1；同时要求CLIENT3访问R1时经过的路径是SW3-SW2-R1; 

##### 可能故障原因：

1. STP 未开或者模式错误

2. MSTP 三要素错误 

3. SW1 和 SW2 主备根桥错误

4.  接口 cost 错误

5. 接口 vlan 划分错误

6. SW 之间 trunk 配置错误

   

##### 解决方案

1. 所有交换机配置 MSTP --- 已知 SW3 错误

```sql
stp enable
stp mode mstp
```

2. 所有交换机配置 MSTP  三要素

```sql
stp region-configuration
	region-name HCIE
	instance 1 vlan 12
 	instance 2 vlan 34
 	active region-configuration
```

3. 配置主备根桥

SW1 配置

```sql
stp instance 1 root primary
stp instance 2 root secondary
```

SW2 配置

```sql
stp instance 1 root secondary
stp instance 2 root primary
```

4. 接口开销值配置错误 -------- 只会发生在SW3上

```sql
interface e0/0/22
	# 假设有
	stp instance 1 cost 10 
```

5. 划分vlan

   SW3

   | 设备 | 接口           | 放行 VLAN |
   | ---- | -------------- | --------- |
   | SW3  | e0/0/1, e0/0/2 | 12        |
   | SW3  | e0/0/3, e0/0/4 | 34        |

6. SW 上接口未放行 vlan

   | 设备 | 接口             | 放行 VLAN |
   | ---- | ---------------- | --------- |
   | SW3  | e0/0/21, e0/0/22 | 2 to 4094 |
   | SW1  | e0/0/21          | 2 to 4094 |
   | SW2  | e0/0/22          | 2 to 4094 |

##### 验证

```sql
[LSW3]disp stp instance 1 brief 
 MSTID  Port                        Role  STP State     Protection
   1    Ethernet0/0/1               DESI  LEARNING        NONE
   1    Ethernet0/0/2               DESI  LEARNING        NONE
   1    Ethernet0/0/21              ROOT  FORWARDING      NONE
   1    Ethernet0/0/22              ALTE  DISCARDING      NONE
[LSW3]disp stp instance 2 brief
 MSTID  Port                        Role  STP State     Protection
   2    Ethernet0/0/3               DESI  FORWARDING      NONE
   2    Ethernet0/0/4               DESI  FORWARDING      NONE
   2    Ethernet0/0/21              ALTE  DISCARDING      NONE
   2    Ethernet0/0/22              ROOT  FORWARDING      NONE
```



#### 3、EBGP选路
 R1访问VLAN12时经过的路径是R1-LSW1-LSW3;访问VLAN34时经过的路径是R1-LSW2-LSW3；
 只允许在AS300中实现，`并且确保你的解决方案不要影响AS100 AS300以外的其他AS`;

##### 分析

1.  只允许在AS300中实现 ---- 只能在 SW1、SW2 上配置
2. 并且确保你的解决方案不要影响AS100 AS300以外的其他AS --- 使用 `med` 值属性 --- 只在相邻 AS 之间传递



![image-20211016234513847](https://i.loli.net/2021/10/16/VqS4yDaI3lzjJF2.png)

##### 可能故障原因

1. RR1 和 SW1、SW2 之间 EBGP 对等体无法建立 (直连接口)

   A. 直连不通  --- SW1 和 SW2 VLAN 划分错误、网段配置错误

   B. EBGP router-id 冲突

   C. EBGP 对等体 AS 号冲突

   D. R1 接口没有绑定实例

   E. R1 基于实例与 SW1、SW2 建立对等体配置错误

2. SW1、SW2 未发布路由

3. SW1、SW2 发布路由，配置错误选路策略

##### 解决方案

​		A. 直连不通

| 设备 | vlan       | ip address   | 对端设备 |
| ---- | ---------- | ------------ | -------- |
| SW1  | VLANIF 100 | 10.1.100.100 | R1       |
| SW2  | VLANIF 200 | 10.1.200.200 | R1       |

SW1、SW2 

```
ping 10.1.100.1
ping 10.1.200.1
```

​		 B. EBGP router-id 冲突

​		 C. EBGP 对等体 AS 号冲突

​		 D. R1 接口没有绑定实例

​		 E. R1 基于实例与 SW1、SW2 建立对等体配置错误

AR1 bgp 配置

```sql
[AR1]bgp 100
[AR1-bgp]disp this
[V200R003C00]
#
bgp 100
 router-id 100.1.1.1
 peer 100.1.1.7 as-number 100 
 peer 100.1.1.7 connect-interface LoopBack0
 #
 ipv4-family unicast
  undo synchronization
  peer 100.1.1.7 enable
 # 
 ipv4-family vpnv4
  policy vpn-target
  peer 100.1.1.7 enable
 #
 ipv4-family vpn-instance 1 
  import-route direct
  peer 10.1.100.100 as-number 300 
  peer 10.1.200.200 as-number 400
```

**可以发现以下问题**

1. vpn-instance 1 下 对 SW2 对等体 AS 号错误

```sql
# 错误
ipv4-family vpn-instance 1 
  	peer 10.1.200.200 as-number 400
 # 正确
 ipv4-family vpn-instance 1 
  	peer 10.1.200.200 as-number 300 
```

AR1 接口配置

```sql
interface GigabitEthernet2/0/1
 	ip address 10.1.100.1 255.255.255.0 
interface GigabitEthernet2/0/2
 	ip binding vpn-instance 1
 	ip address 10.1.200.1 255.255.255.0 
```

**可以发现 g2/0/1 接口未绑定 vpn 实例**

```sql
interface GigabitEthernet2/0/1
 	ip binding vpn-instance 1
 	ip address 10.1.100.1 255.255.255.0 
```

SW1 bgp 配置

```sql
bgp 300
 	router-id 10.1.200.200
 	peer 10.1.100.1 as-number 100
 #
 	ipv4-family unicast
    undo synchronization
    network 10.1.12.0 255.255.255.0 route-policy AS
    network 10.1.34.0 255.255.255.0 route-policy MED
    peer 10.1.100.1 enable
```

SW2 bgp 配置

```sql
bgp 300
 	peer 10.1.200.1 as-number 100
 #
 	ipv4-family unicast
    undo synchronization
    network 10.1.12.0 255.255.255.0
    network 10.1.34.0 255.255.255.0 route-policy MED
    peer 10.1.200.1 enable
```

**可以发现以下问题：**

1. SW1 bgp.router-id 错误

```sql
# 改正
bgp 300
router-id 10.1.100.100
Warning: Changing the parameter in this command resets the peer session. Continu
e?[Y/N]:y
```

2. SW2 bgp.router-id 未配置

```sql
bgp 300
	router-id 10.1.200.200
```

##### 验证 1

```sql
disp bgp vpnv4 all peer
```

2. SW1、SW2 未发布路由

可以看到有发布路由

3. SW1、SW2 发布路由，配置错误选路策略

SW1 发布路由策略错误

```sql
# 错误
ipv4-family unicast
	network 10.1.12.0 255.255.255.0 route-policy AS
  network 10.1.34.0 255.255.255.0 route-policy MED
```

SW1 是对 `10.1.12.0/24` 正常发布，`10.1.12.0/24` 修改 `MED` 值

改正如下：

```sql
# 正确
ipv4-family unicast
	network 10.1.12.0 255.255.255.0
  network 10.1.34.0 255.255.255.0 route-policy MED
```

查看 route-policy med --- 发现是通过把 `med` 值改大

```sql
route-policy MED permit node 1
	apply cost 100
```

SW2 发布路由策略错误 

SW2 是对 `10.1.12.0/24` 修改 `MED` 值，`10.1.12.0/24` 正常发布

正确如下

```SQL
ipv4-family unicast
  network 10.1.12.0 255.255.255.0 route-policy MED
  network 10.1.34.0 255.255.255.0
```

##### 验证 2

R1

```sql
[AR1]disp bgp vpnv4 all routing-table 
	*>   10.1.12.0/24       10.1.100.100    0                     0      300i
 	*                       10.1.200.200    100                   0      300i
	*>   10.1.34.0/24       10.1.200.200    0                     0      300i
 	*                       10.1.100.100    100                   0      300i
```

tracert 看一下 --- 可以看到没问题

```sql
[AR1]tracert -vpn-instance 1 10.1.12.11 # 访问 PC1

 traceroute to 1 10.1.12.11(10.1.12.11), max hops: 30 ,packet length: 40,press CTRL_C to break 

 1 10.1.100.100 40 ms  20 ms  20 ms 
 2 10.1.12.11 60 ms  70 ms  60 ms 
 
[AR1]tracert -vpn-instance 1 10.1.34.33   # 访问 RC4
 traceroute to 1 10.1.34.33(10.1.34.33), max hops: 30 ,packet length: 40,press CTRL_C to break 

 1 10.1.200.200 20 ms 20 ms  20 ms 
 2 10.1.34.33 110 ms  60 ms  80 ms 
```

------



#### 4、MUX-VLAN
 Site4中，AR24、AR25、AR26在一个网段中，同时都运行了ISIS协议，要求AR26能和AR24、AR25都能形成邻居关系，**但是AR24与AR25不能形成邻居关系；**通过LSW8的二层VLAN技术以及其他设备排除错误点来实现此要求， 

注意；配置过程中不能在LSW8上删除和增加新的VLAN

![image-20211017141901761](https://i.loli.net/2021/10/17/tvoLrwkahfTW3mG.png)

##### 可能故障原因

1. 交换机 `MUX-VLAN`、`VLAN`问题
   1. 交换机 `MUX-VLAN` 划分错误
   2. 交换机接口 `MUX-VLAN` 未开启
   3. 交换机接口 VLAN 划分错误
2. IS-IS 邻居无法建立
   1. 全部采用 Level-2，层级配置错误
   2. system-id 冲突
   3. 接口认证错误
   4. 接口未开启 isis 功能

------

##### 解决方案

一、交换机 `MUX-VLAN`、`VLAN`问题

> ​	1. 检查 `MUX-VLAN` SW8 配置
>

```sql
[LSW8]vlan 100
[LSW8-vlan100]disp this
vlan 100
	mux-vlan
  subordinate group 50
```

改正为

```sql
vlan 100
	undo subordinate group 50
  subordinate separate 50
```

> 2. 检查接口 VLAN 划分以及是否开启 MUX-VLAN

```sql
interface GigabitEthernet0/0/1
  port link-type access
  port default vlan 50
  port mux-vlan enable  	# 未开
interface GigabitEthernet0/0/1
	port link-type access
	port default vlan 50
	port mux-vlan enable	# 未开
interface GigabitEthernet0/0/3
 	port link-type access
 	port default vlan 100   # 划分错误为 50
 	port mux-vlan enable   # 未开
```

> 3. 验证结果

![image-20211018135927239](https://i.loli.net/2021/10/18/NYJV7C2lebdwfFT.png)

```sql
或者 disp vlan
```

![image-20211119165330006](https://i.loli.net/2021/11/19/4ysZYrOPniEHqxQ.png)

------

二、IS-IS 邻居无法建立

**检查 isis 配置**

> 1. 检查 R26

```sql
disp cu con isis # 查看进程号
isis 100
 	is-level level-1
 	network-entity 47.0004.0000.0000.0026.00 
```

发现进程号为 100，且 等级配置错误、区域号配置错误， 进行改正

```sql
isis 100
 	is-level level-2
 	network-entity 49.0004.0000.0000.0026.00
```

> 2. 检查 R24

```sql
<AR24>disp cu con isis
[V200R003C00]
isis 100
 is-level level-2
 network-entity 49.0004.0000.0000.0026.00
```

发现 `system-id` 配置错误，改正如下

```sql
isis 100
 is-level level-2
 network-entity 49.0004.0000.0000.0024.00
```

**检查 isis 接口 以及 接口认证**

```sql
disp isis int
```

R24、R25、R26 配置认证

```sql
int g0/0/0
	isis enable 100
 	isis authentication-mode md5 cipher hcie   # 具体密码看考场上题本
```

验证

![image-20211018142256634](https://i.loli.net/2021/10/18/z8srySnbidgjpk1.png)

隐含需求

上图中可以看出 R24 是 DIS、他会周期性发送 `CSNP` 报文来同步链路数据库信息，但是因为`隔离型 vlan`无法发送给 R25，所以 R24 不能成为 DIS，得需要 R26 成为 DIS

![image-20211018142535338](https://i.loli.net/2021/10/18/WtPj9d5kT4zV6KD.png)

R26

```sql
int g0/0/0
	isis dis-priority 127
```

![image-20211018142845298](https://i.loli.net/2021/10/18/62VHWXUlKEQPbOD.png)

------

#### 5、MPLS-VPN实现site1-site4互访

 Site1与Site4为同一个VPN客户的两个站点，现在site1里的CLIENTS无法和site4里的CLIENT通信，请解决此问题； 
$\textcolor{red}{注意：不要删除现有配置，可修改解决}$

考察 OPTION B

> 1. 直接互传 VPNV4 路由
> 2. 开启的是 MP-BGP 的能力
> 3. 不需要维护实例

##### 可能故障原因：

1. 控制层面传递路由故障
   1. PE设备(R23) ISIS-BGP 双向引入错误 --- R1 已经在选路的时候解决了
   2. VPNV4 对等体建立错误
   3. RR 路由反射器客户端配置错误
   4. 所有设备（除 PE 设备）都需要参与到`VPNV4`路由传递，未关闭 `RT`值检测
   5. PE 设备 `RT` 值配置错误
2. 转发层面 --- 底层 LSP 没有成功建立
   1. 底层 IGP 存在问题
   2. LDP 会话没有建立
3. ASBR 之间接口没有开启 `MPLS` 能力

![image-20211018155318855](https://i.loli.net/2021/10/18/U6M9RJ3sdy4Nx1w.png)

##### 解决方案

###### 1. 检查AR23的BGP VPNv4路由----双向引入是否正确

```sql
disp bgp vpnv4 all peer
  Peer            V          AS  MsgRcvd  MsgSent  OutQ  Up/Down       State PrefRcv
  200.1.1.9       4         200      608      591     0 09:38:45 Established    0
disp bgp vpnv4 all routing
 VPN-Instance 1, Router ID 200.1.1.23:

 Total Number of Routes: 7
      Network            NextHop        MED        LocPrf    PrefVal Path/Ogn

 *>   10.4.1.0/24        0.0.0.0         10                    0      ?
 *>   10.4.1.23/32       0.0.0.0         0                     0      ?
 *>   10.4.1.25/32       0.0.0.0         10                    0      ?
 *>   10.4.1.26/32       0.0.0.0         20                    0      ?
 *>   10.4.27.0/24       0.0.0.0         30                    0      ?
 *>   10.4.128.0/24      0.0.0.0         0                     0      ?
 *>   10.4.129.0/24      0.0.0.0         20                    0      ?
```

发现有 Site4 PC 网段地址 `10.4.27.0/24`

```sql
bgp 200
	ipv4-family vpn-instance 1 
  	import-route isis 100
```

```sql
isis 100 vpn-instance 1
	is-level level-2
  network-entity 47.0004.0000.0000.0023.00
  import-route bgp  # 加上此行
```

###### 2. 检查所有路由的 VPNV4 邻居关系

R9(23、4、5)、R2（4、5、7）、R7 （1、2、6、13）检查

```sql
disp bgp vpnv4 all peer  # 没有错误
```

###### 3. R7 和 R9 作为 VPNv4 路由反射器，配置正确客户端

R7 配置

```sql
bgp 200
ipv4-family vpnv4
	policy vpn-target
  peer 100.1.1.1 enable
  peer 100.1.1.1 reflect-client
  peer 100.1.1.2 enable
  peer 100.1.1.2 reflect-client
```

R9 配置

```sql
bgp 100
ipv4-family vpnv4
	undo policy vpn-target
  peer 200.1.1.4 enable
  peer 200.1.1.4 reflect-client
  peer 200.1.1.5 enable
  peer 200.1.1.5 reflect-client
  peer 200.1.1.23 relect-client
```

###### 4. RR、ASBR 关闭 RT 检测、PE(R1、R23) 开启 RT 值检测

 AR7、AR2、AR4、AR5、AR9

<img src="https://i.loli.net/2021/10/18/IAnrxg6J7YpGSzE.png" alt="image-20211018212055219" style="zoom:50%;" />

```sql
ipv4-family vpnv4
	undo policy vpn-target
```

R1、R23

```sql
ipv4-family vpnv4
	policy vpn-target
```

###### 5. 两端 PE 的 `RT` 配置错误

![image-20211018213411706](https://i.loli.net/2021/10/18/aT8Xbefc1QGSPlI.png)

![image-20211018213439642](https://i.loli.net/2021/10/18/TtD2lpg7jNsLJYE.png)

R1 修改 RT 值 --- 不删原来的，加一下就行了

```sql
ip vpn-instance 1
  vpn-target 200:100 both 
```

R23 上查看 vpnv4 路由，发现有 `10.1.12.0/24 ` 和 `10.1.34.0/24`，但是不是最优的

```sql
[AR23]disp bgp vpnv4 all routing-table 
 Route Distinguisher: 200:100 
      Network            NextHop        MED        LocPrf    PrefVal Path/Ogn
 *>i  10.1.1.1/32        200.1.1.4                  100        0      100?
 *>i  10.1.12.0/24       200.1.1.4                  100        0      100 300i
 *>i  10.1.34.0/24       200.1.1.4                  100        0      100 300i
 *>i  10.1.100.0/24      200.1.1.4                  100        0      100?
 *>i  10.1.200.0/24      200.1.1.4                  100        0      100?
 *>   10.4.1.0/24        0.0.0.0         10                    0      ?
 *>   10.4.1.23/32       0.0.0.0         0                     0      ?
 *>   10.4.1.25/32       0.0.0.0         10                    0      ?
 *>   10.4.1.26/32       0.0.0.0         20                    0      ?
 *>   10.4.27.0/24       0.0.0.0         30                    0      ?
 *>   10.4.128.0/24      0.0.0.0         0                     0      ?
 *>   10.4.129.0/24      0.0.0.0         20                    0      ?

 VPN-Instance 1, Router ID 200.1.1.23:
 Total Number of Routes: 8
      Network            NextHop        MED        LocPrf    PrefVal Path/Ogn
   i  10.1.12.0/24       200.1.1.4                  100        0      100 300i # 不是最优
 *>   10.4.1.0/24        0.0.0.0         10                    0      ?
 *>   10.4.1.23/32       0.0.0.0         0                     0      ?
 *>   10.4.1.25/32       0.0.0.0         10                    0      ?
 *>   10.4.1.26/32       0.0.0.0         20                    0      ?
 *>   10.4.27.0/24       0.0.0.0         30                    0      ?
 *>   10.4.128.0/24      0.0.0.0         0                     0      ?
 *>   10.4.129.0/24      0.0.0.0         20                    0      ?
```

R4 上查看自己端路由，发现是最优的

```sql
<AR4>disp bgp vpnv4 all routing-table 
Route Distinguisher: 200:100 

      Network            NextHop        MED        LocPrf    PrefVal Path/Ogn

 *>   10.1.1.1/32        200.100.24.2                          0      100?
 *>   10.1.12.0/24       200.100.24.2                          0      100 300i
 *>   10.1.34.0/24       200.100.24.2                          0      100 300i
 *>   10.1.100.0/24      200.100.24.2                          0      100?
 *>   10.1.200.0/24      200.100.24.2                          0      100?
 *>i  10.4.1.0/24        200.1.1.23      10         100        0      ?
 *>i  10.4.1.23/32       200.1.1.23      0          100        0      ?
 *>i  10.4.1.25/32       200.1.1.23      10         100        0      ?
 *>i  10.4.1.26/32       200.1.1.23      20         100        0      ?
 *>i  10.4.27.0/24       200.1.1.23      30         100        0      ?
 *>i  10.4.128.0/24      200.1.1.23      0          100        0      ?
 *>i  10.4.129.0/24      200.1.1.23      20         100        0      ?
```

查看是否通告给 R2 --- 发现并没有通告

```sql
<AR4>disp bgp vpnv4 all routing-table peer 200.100.24.2 advertised-routes 
```

原因是因为``没有分配``去往 `200.1.1.23` 的 lsp

```sql
<AR4>disp mpls lsp
-------------------------------------------------------------------------------
                 LSP Information: L3VPN  LSP
-------------------------------------------------------------------------------
FEC                In/Out Label  In/Out IF                      Vrf Name       
10.1.12.0/24       1025/1031     -/-                            ASBR LSP       
100.1.136.6/32     1026/1032     -/-                            ASBR LSP       
10.2.1.11/32       1027/1033     -/-                            ASBR LSP       
10.2.1.10/32       1028/1034     -/-                            ASBR LSP       
10.2.116.0/24      1029/1035     -/-                            ASBR LSP       
10.2.106.0/24      1030/1036     -/-                            ASBR LSP       
10.2.129.0/24      1031/1037     -/-                            ASBR LSP       
10.23.1.0/24       1032/1038     -/-                            ASBR LSP       
10.3.1.20/32       1033/1039     -/-                            ASBR LSP       
10.3.128.0/24      1034/1040     -/-                            ASBR LSP       
10.3.1.14/32       1035/1041     -/-                            ASBR LSP       
10.3.1.17/32       1036/1042     -/-                            ASBR LSP       
10.3.147.0/24      1037/1043     -/-                            ASBR LSP       
10.3.1.15/32       1038/1044     -/-                            ASBR LSP       
10.3.210.0/24      1039/1045     -/-                            ASBR LSP       
10.3.1.19/32       1040/1046     -/-                            ASBR LSP       
10.3.192.0/24      1041/1047     -/-                            ASBR LSP       
10.3.159.0/24      1042/1048     -/-                            ASBR LSP       
100.1.136.13/32    1043/1049     -/-                            ASBR LSP       
10.3.1.13/32       1044/1050     -/-                            ASBR LSP       
10.3.157.0/24      1045/1051     -/-                            ASBR LSP       
10.2.128.0/24      1046/1052     -/-                            ASBR LSP       
10.1.34.0/24       1047/1053     -/-                            ASBR LSP       
10.1.100.0/24      1048/1054     -/-                            ASBR LSP       
10.1.200.0/24      1049/1055     -/-                            ASBR LSP       
10.1.1.1/32        1050/1056     -/-                            ASBR LSP       
-------------------------------------------------------------------------------
                 LSP Information: LDP LSP
-------------------------------------------------------------------------------
FEC                In/Out Label  In/Out IF                      Vrf Name       
200.1.1.4/32       3/NULL        -/-                                           
200.1.1.5/32       NULL/3        -/GE0/0/0                                     
200.1.1.5/32       1024/3        -/GE0/0/0                                     
```

###### 6. 部分设备 LDP 会话未建立

```sql
<AR4>disp mpls ldp session all

 LDP Session(s) in Public Network
 Codes: LAM(Label Advertisement Mode), SsnAge Unit(DDDD:HH:MM)
 A '*' before a session means the session is being deleted.
 ------------------------------------------------------------------------------
 PeerID             Status      LAM  SsnRole  SsnAge      KASent/Rcv
 ------------------------------------------------------------------------------
 200.1.1.99:0       NonExistent      Passive              0/0    # 未建立成功
 200.1.1.5:0        Operational DU   Passive  0000:08:07  1951/1951
 ------------------------------------------------------------------------------
 TOTAL: 2 session(s) Found.

Oct 18 2021 21:52:30-08:00 AR4 %%01LDP/4/SSNHOLDTMREXP(l)[0]:Sessions were delet
ed because the session hold timer expired and the notification of the expiry was
 sent to the peer 200.1.1.99. 

```

看一下是不是 环回口不可达 --- 果然是

```sql
<AR4>ping -a 200.1.1.4 200.1.1.5
  PING 200.1.1.5: 56  data bytes, press CTRL_C to break
    Reply from 200.1.1.5: bytes=56 Sequence=1 ttl=255 time=30 ms
    Reply from 200.1.1.5: bytes=56 Sequence=2 ttl=255 time=20 ms
    Reply from 200.1.1.5: bytes=56 Sequence=3 ttl=255 time=20 ms
    Reply from 200.1.1.5: bytes=56 Sequence=4 ttl=255 time=10 ms
    Reply from 200.1.1.5: bytes=56 Sequence=5 ttl=255 time=20 ms

  --- 200.1.1.5 ping statistics ---
    5 packet(s) transmitted
    5 packet(s) received
    0.00% packet loss
    round-trip min/avg/max = 10/20/30 ms

<AR4>ping -a 200.1.1.4 200.1.1.99  # 不可达
  PING 200.1.1.99: 56  data bytes, press CTRL_C to break
    Request time out
    Request time out
    Request time out
    Request time out
    Request time out
```

然后得去 R9 上找原因

> 1. 查看是否 200.1.1.99 是哪个接口
>
>    发现是 Loop1 接口

```sql
[AR9]disp ip int brief 

Interface                         IP Address/Mask      Physical   Protocol  
GigabitEthernet0/0/0              200.1.239.9/24       up         up        
GigabitEthernet0/0/1              200.1.49.9/24        up         up        
GigabitEthernet0/0/2              200.1.59.9/24        up         up        
LoopBack0                         200.1.1.9/32         up         up(s)     
LoopBack1                         200.1.1.99/32        up         up(s)     
NULL0                             unassigned           up         up(s)     
Serial2/0/0                       unassigned           down       down      
Serial2/0/1                       unassigned           down       down      
Serial3/0/0                       200.1.209.9/24       up         down      
Serial3/0/1                       unassigned           down       down      
```

> 2. 查看是否宣告进 isis
>
>    发现 Loop1 并没有宣告进 isis

```sql
[AR9]disp isis int
                      Interface information for ISIS(200)
                      -----------------------------------
 Interface       Id      IPV4.State          IPV6.State      MTU  Type  DIS   
 GE0/0/0         001         Up                  Up          1497 L1/L2 No/Yes
 GE0/0/1         002         Up                  Up          1497 L1/L2 No/Yes
 GE0/0/2         003         Up                  Up          1497 L1/L2 No/No 
 Loop0           001         Up                  Up          1500 L1/L2 -- 
```

> 3. 宣告进 isis

```sql
interface LoopBack1
	isis enable 200
```

> 4. R9 查看 ldp session 会话 
>
>    可以看到都已经建立完成了

```sql
[AR9]disp mpls ldp session all

 LDP Session(s) in Public Network
 Codes: LAM(Label Advertisement Mode), SsnAge Unit(DDDD:HH:MM)
 A '*' before a session means the session is being deleted.
 ------------------------------------------------------------------------------
 PeerID             Status      LAM  SsnRole  SsnAge      KASent/Rcv
 ------------------------------------------------------------------------------
 200.1.1.4:0        Operational DU   Active   0000:00:03  15/15
 200.1.1.5:0        Operational DU   Active   0000:00:03  14/14
 200.1.1.23:0       Operational DU   Active   0000:00:03  15/15
 ------------------------------------------------------------------------------
 TOTAL: 3 session(s) Found.
```

> 5. 现在在 R4 上检查 mpls 和 通告路由
>
>    发现有去往 `200.1.1.23/32` 的 lsp 了
>
>    发现 R4 路由已经通告给 R2 了

```sql
<AR4>disp mpls lsp
-------------------------------------------------------------------------------
                 LSP Information: L3VPN  LSP
-------------------------------------------------------------------------------
FEC                In/Out Label  In/Out IF                      Vrf Name           
-------------------------------------------------------------------------------
                 LSP Information: LDP LSP
-------------------------------------------------------------------------------
FEC                In/Out Label  In/Out IF                      Vrf Name       
200.1.1.4/32       3/NULL        -/-                                           
200.1.1.5/32       NULL/3        -/GE0/0/0                                     
200.1.1.5/32       1024/3        -/GE0/0/0                                     
200.1.1.99/32      NULL/3        -/GE0/0/1                                     
200.1.1.99/32      1051/3        -/GE0/0/1                                     
200.1.1.9/32       NULL/3        -/GE0/0/1                                     
200.1.1.9/32       1052/3        -/GE0/0/1                                     
200.1.1.23/32      NULL/1025     -/GE0/0/1                                     
200.1.1.23/32      1053/1025     -/GE0/0/1                                     
```

```sql
<AR4>disp bgp vpnv4 all routing-table peer 200.100.24.2 advertised-routes 
 Total Number of Routes: 7
 Route Distinguisher: 200:100 
      Network            NextHop        MED        LocPrf    PrefVal Path/Ogn

 *>i  10.4.1.0/24        200.100.24.4                          0      200?
 *>i  10.4.1.23/32       200.100.24.4                          0      200?
 *>i  10.4.1.25/32       200.100.24.4                          0      200?
 *>i  10.4.1.26/32       200.100.24.4                          0      200?
 *>i  10.4.27.0/24       200.100.24.4                          0      200?
 *>i  10.4.128.0/24      200.100.24.4                          0      200?
 *>i  10.4.129.0/24      200.100.24.4                          0      200?
```

> 7. ASBR 之间接口 务必开启 `MPLS` 能力

R2 查看标签 --- 发现接收到的 `10.4.27.0 ...`我好不容易传来的路由没有分配标签

```sql
<AR2>disp bgp vpnv4 all routing-table label

 Route Distinguisher: 200:100 
        Network           NextHop           In/Out Label

 *>i    10.1.1.1          100.1.1.1         1056/1031
 *>i    10.1.12.0         100.1.1.1         1031/1035
 *>i    10.1.34.0         100.1.1.1         1053/1036
 *>i    10.1.100.0        100.1.1.1         1054/1037
 *>i    10.1.200.0        100.1.1.1         1055/1034
 *>     10.4.1.0          200.100.24.4      NULL/1057
 *                        200.100.25.5      NULL/1057
 *>     10.4.1.23         200.100.24.4      NULL/1060
 *                        200.100.25.5      NULL/1060
 *>     10.4.1.25         200.100.24.4      NULL/1058
 *                        200.100.25.5      NULL/1058
 *>     10.4.1.26         200.100.24.4      NULL/1055
 *                        200.100.25.5      NULL/1055
 *>     10.4.27.0         200.100.24.4      NULL/1054
 *                        200.100.25.5      NULL/1054
 *>     10.4.128.0        200.100.24.4      NULL/1059
 *                        200.100.25.5      NULL/1059
 *>     10.4.129.0        200.100.24.4      NULL/1056
 *                        200.100.25.5      NULL/1056
```

> 检查发现 R2 的接口下未开启 MPLS 标签能力，则会导致无法继续为 VPNv4 分配私网标签传递给 AR7（RR）

```sql
<AR2>disp mpls int
Interface             Status    TE Attr   LSP Count  CRLSP Count Effective MTU
GE0/0/0               Up        Dis       2          0           1500      
GE0/0/1               Up        Dis       4          0           1500      
GE0/0/2               Up        Dis       8          0           1500   
```

![image-20211018224507361](https://i.loli.net/2021/10/18/jNxEwBKZTtbdWXc.png)

> 现在查看 R2 的 mpls label，发现已经可以分配标签了
>

```sql
[AR2]disp bgp vpnv4 all rou label  
 Route Distinguisher: 200:100 
        Network           NextHop           In/Out Label

 *>i    10.1.1.1          100.1.1.1         1056/1031
 *>i    10.1.12.0         100.1.1.1         1031/1035
 *>i    10.1.34.0         100.1.1.1         1053/1036
 *>i    10.1.100.0        100.1.1.1         1054/1037
 *>i    10.1.200.0        100.1.1.1         1055/1034
 *>     10.4.1.0          200.100.24.4      1060/1057
 *                        200.100.25.5      NULL/1057
 *>     10.4.1.23         200.100.24.4      1057/1060
 *                        200.100.25.5      NULL/1060
 *>     10.4.1.25         200.100.24.4      1059/1058
 *                        200.100.25.5      NULL/1058
 *>     10.4.1.26         200.100.24.4      1062/1055
 *                        200.100.25.5      NULL/1055
 *>     10.4.27.0         200.100.24.4      1063/1054
 *                        200.100.25.5      NULL/1054
 *>     10.4.128.0        200.100.24.4      1058/1059
 *                        200.100.25.5      NULL/1059
 *>     10.4.129.0        200.100.24.4      1061/1056
 *                        200.100.25.5      NULL/1056

```

> 8. Site4 --- Site1 pc 互访测试

![image-20211019000734138](https://i.loli.net/2021/10/19/xaOtihVMuqYN26f.png)

可以发现 ping 通 pc1，但是 ping 不通 pc3 那是怎么回事呢？

1. 可以排除是 跨域错误，不然 pc1 也不可能通的
2. 可以在 R23、R1 上查一查

> 1. 查看 vpnv4 路由 --- 发现有 12，34 的路由

```sql
[AR23]disp bgp vpnv4 all routing-table

 Total number of routes from all PE: 12
 Route Distinguisher: 200:100 

      Network            NextHop        MED        LocPrf    PrefVal Path/Ogn

 *>i  10.1.1.1/32        200.1.1.4                  100        0      100?
 *>i  10.1.12.0/24       200.1.1.4                  100        0      100 300i  # 这里
 *>i  10.1.34.0/24       200.1.1.4                  100        0      100 300i  # 这里
 *>i  10.1.100.0/24      200.1.1.4                  100        0      100?
 *>i  10.1.200.0/24      200.1.1.4                  100        0      100?
 *>   10.4.1.0/24        0.0.0.0         10                    0      ?
 *>   10.4.1.23/32       0.0.0.0         0                     0      ?
 *>   10.4.1.25/32       0.0.0.0         10                    0      ?
 *>   10.4.1.26/32       0.0.0.0         20                    0      ?
 *>   10.4.27.0/24       0.0.0.0         30                    0      ?
 *>   10.4.128.0/24      0.0.0.0         0                     0      ?
 *>   10.4.129.0/24      0.0.0.0         20                    0      ?

 VPN-Instance 1, Router ID 200.1.1.23:
 Total Number of Routes: 8
      Network            NextHop        MED        LocPrf    PrefVal Path/Ogn

 *>i  10.1.12.0/24       200.1.1.4                  100        0      100 300i # 这里
 *>   10.4.1.0/24        0.0.0.0         10                    0      ?
 *>   10.4.1.23/32       0.0.0.0         0                     0      ?
 *>   10.4.1.25/32       0.0.0.0         10                    0      ?
 *>   10.4.1.26/32       0.0.0.0         20                    0      ?
 *>   10.4.27.0/24       0.0.0.0         30                    0      ?
 *>   10.4.128.0/24      0.0.0.0         0                     0      ?
 *>   10.4.129.0/24      0.0.0.0         20                    0      ?

```

> 2. 那就奇了怪了，但是仔细发现  VPN-Instance 1 下 只有 12，没有 34 路由
>
>    所以 R23 检查一下 vpn-instance 看看

```sql
ip vpn-instance 1
	description site1-site4
  service-id 14
  ipv4-family
    route-distinguisher 200:100
    import route-policy import   # 这里有个入向策略
    vpn-target 200:100 export-extcommunity
    vpn-target 200:100 import-extcommunity
```

> 3. 发现一个入向策略，我们查看一下

```sql
route-policy import permit node 1 
	if-match acl 2000 
```

> 4. 在追查一下 acl 2000

```sql
acl number 2000  
	rule 1 permit source 10.1.12.0 0 
```

> 5. 好家伙呢？只匹配了 10.1.12.0，藏得够深啊，怪不得呢？
>
>    现在不删除原有配置情况下，加上 10.1.34.0 就好了

```sql
 rule 5 permit source 10.1.34.0 0 
```

> 6. R23 检查 vpn-instance 路由

```sql
[AR23]disp ip rou vpn-instance 1
Route Flags: R - relay, D - download to fib
------------------------------------------------------------------------------
Routing Tables: 1
         Destinations : 12       Routes : 15       

Destination/Mask    Proto   Pre  Cost      Flags NextHop         Interface

      10.1.12.0/24  IBGP    255  0          RD   200.1.1.4       GigabitEthernet0/0/0
      10.1.34.0/24  IBGP    255  0          RD   200.1.1.4       GigabitEthernet0/0/0

```

> 7. pc 互访看下，可以 ping 通了
> 8. OPTION-B tracert 有断层正常

![image-20211019001852673](https://i.loli.net/2021/10/19/dKpgHrN7OQhl3ki.png)

------

##### 总结：

1. 考场上频繁发生的，ASBR 没有开启 `mpls` 能力
2. R9 上环回口没有宣告进 isis
3. R23 没有把路由从 bgp 引入进 isis
4. 一直再强调哪些错误的可能性是什么

   1. 你 R2 上什么路由都没有的话，可能性最大的就是没有关闭 `RT` 值检测
   2. 如果只有本端没有对端的，是不是对端和你的对等体没有建立好，或者对端的域内出现一些问题

------



#### 6、 OSPF 邻居

 AS100中所部分设备的邻居关系有问题，解决此问题

![image-20211019002333516](https://i.loli.net/2021/10/19/tsOhBPbrvi3G7EI.png)

##### 检查命令

1. `disp ospf peer brief`
2. `disp ospf error int g0/0/0` 
3. `disp ospf int`

------

#### 7、VRRP

 Site2中AR10与AR11要为LSW4的PC4提供第一跳网关冗余服务，虚拟网关地址为10.2.129.254和10.2.129.253；为了加速VRRP的收敛， 使用BFD跟踪上行链路状态以及VRRP的邻居关系；

##### 分析

![image-20211019121054474](https://i.loli.net/2021/10/19/bEuBVc7yHXsI9Sn.png)

准备好两个 虚拟路由器作为网关`10.2.129.254`, `10.2.129.253`，R10 作为 vrid 1 的 master，R11 作为 vird 2 的 master

准备好两个 `dhcp server`，分别维护同一个网段 `10.2.129.0/24`, 但是下发的网关不同，一个是 V1、另一个是 V2

------

##### 解决方案

###### 一、配置 BFD

1. R10 配置

```sql
bfd
q
bfd 1 bind peer-ip 10.2.106.6 source-ip 10.2.106.10 auto
commit
q
bfd 2 bind peer-ip 10.2.129.11 source-ip 10.2.129.10 auto
commit
```

2. R11 配置

```sql
bfd
q
bfd 1 bind peer-ip 10.2.116.6 source-ip 10.2.116.11 auto
commit
q
bfd 2 bind peer-ip 10.2.129.10 source-ip 10.2.129.11 auto
commit
```

3. R106 配置

```sql
bfd 1 bind peer-ip 10.2.106.10 source-ip 10.2.106.6 auto
commit
q
bfd 2 bind peer-ip 10.2.116.11 source-ip 10.2.116.6 auto
commit
```

4. 验证

R10、R11、R106

```sql
disp bfd session all   # 双双 up 就好了
```

------

###### 二、配置 VRRP

1. R10 配置

```sql
interface GigabitEthernet0/0/0
	vrrp vrid 1 virtual-ip 10.2.129.254
  vrrp vrid 1 priority 200
  vrrp vrid 1 preempt-mode timer delay 1
  vrrp vrid 1 track bfd-session session-name 1 reduced 120     # 易错 加上
  vrrp vrid 1 authentication-mode md5 hcie

  vrrp vrid 2 virtual-ip 10.2.129.253
  vrrp vrid 2 preempt-mode timer delay 1
  vrrp vrid 2 track bfd-session session-name 2 increase 120
  vrrp vrid 2 authentication-mode md5 hcie
```

2. R11 配置

```sql
interface GigabitEthernet0/0/0
	vrrp vrid 1 virtual-ip 10.2.129.254
  vrrp vrid 1 preempt-mode timer delay 1
  vrrp vrid 1 track bfd-session session-name 2 increase 120
  vrrp vrid 1 authentication-mode md5 hcie

  vrrp vrid 2 virtual-ip 10.2.129.253
  vrrp vrid 2 priority 200
  vrrp vrid 2 preempt-mode timer delay 1
  vrrp vrid 2 track bfd-session session-name 1 reduced 120
  vrrp vrid 2 authentication-mode md5 hcie
```

3. 验证

`disp vrrp 1`

`disp vrrp brief`

注意：部分配置不会直接覆盖，建议删除重新配置

![image-20211019131110204](https://i.loli.net/2021/10/19/DEro8hgBCdntX9m.png)

![image-20211019131044905](https://i.loli.net/2021/10/19/sV3AyeZGPCSRrE7.png)

------

#### 8、DHCP

 Site2中，AR10 AR11是DHCP服务器并且相互备份，要求CLIENT7能通过DHCP服务器获取到地址10.2.129.100；
 要求CLIENT8只能获取指定地址为10.2.129.101；现在CLIENT8有时无法获取地址，请解决； 

##### 一、配置 IP 地址池

1. R10 配置

```sql
ip pool HCIE
  gateway-list 10.2.129.254
  network 10.2.129.0 mask 255.255.255.0
  excluded-ip-address 10.2.129.1 10.2.129.99		# 加上
  static-bind ip-address 10.2.129.100 mac-address 5489-9833-3389
  static-bind ip-address 10.2.129.101 mac-address 5489-9812-4E45
  dns-list 8.8.8.8
  domain-name huawei.com
```

2. R11 配置

```sql
ip pool HCIE
  gateway-list 10.2.129.253
  network 10.2.129.0 mask 255.255.255.0 
  excluded-ip-address 10.2.129.1 10.2.129.99 
  static-bind ip-address 10.2.129.100 mac-address 5489-9833-3389
  static-bind ip-address 10.2.129.101 mac-address 5489-9812-4E45 
  dns-list 8.8.8.8 
  domain-name huawei.com
```

##### 二、配置接口 `dhcp select global`

R10、R11

```sql
int g0/0/0
	dhcp select global
```

##### 三、可能配置 `dhcp snooping trusted`

SW4

```sql
vlan 1
  dhcp snooping enable
  dhcp snooping trusted interface Ethernet0/0/1
  dhcp snooping trusted interface Ethernet0/0/3
```

##### 四、验证 pc 的 dhcp 地址

```sql
ipconfig /renew
# 如果考试刷不出来就不要刷了，接着往后面做
```

1. PC7

![image-20211019134146254](https://i.loli.net/2021/10/19/dXoma9C6GMpyVFO.png)

2. PC8

![image-20211019134213739](https://i.loli.net/2021/10/19/MFNCaQ1V45DIpYq.png)

------

#### 9、VRRP6

 正确部署VRRP6，要求AR10成为Master设备

##### 解决方案

R10

```sql
interface GigabitEthernet0/0/0
  ipv6 enable 
  ip address 10.2.129.10 255.255.255.0 
  ipv6 address 2002:10:2:129::10/64 
  ipv6 address FE80::10 link-local
  vrrp6 vrid 1 virtual-ip FE80::254 link-local
  vrrp6 vrid 1 virtual-ip 2002:10:2:129::254
```

R11

```sql
interface GigabitEthernet0/0/0
  ipv6 enable 
  ip address 10.2.129.11 255.255.255.0 
  ipv6 address 2002:10:2:129::254/64 # 这里的 ipv6 地址
  ipv6 address FE80::11 link-local
  vrrp6 vrid 1 virtual-ip FE80::254 link-local
  vrrp6 vrid 1 virtual-ip 2002:10:2:129::254	# 与 vrrp6 地址一致
```

会导致，优先级变为 `255`

即使 R10 配置了，也不会成为 master

```sql
int g0/0/0
	vrrp6 vrid 1 priority 120
```

R11 上查看

```sql
disp vrrp6
```

![image-20211019152731471](https://i.loli.net/2021/10/19/pSfNOWqJRFn3rXy.png)

因为 R11 的 ipv6 地址，与 vrrp6 的 `virtual-ip` 冲突，所以更改 ipv6 地址

```sql
int g0/0/0
	undo ipv6 address 2002:10:2:129::254/64
	# 此时 vrrp6 vrid 1 virtual-ip 2002:10:2:129::254 也会消失
	ipv6 address 2002:10:2:129::11 64
	vrrp6 vrid 1 virtual-ip 2002:10:2:129::254
```

##### 验证

注意：考场观察一下新增 client14 的网关，虚拟IP地址与网关地址一致

R10

![image-20211019153145140](https://i.loli.net/2021/10/19/p158D9duXZjyfHT.png)

R11

![image-20211019153205059](https://i.loli.net/2021/10/19/zlgdt2XI5vAoLeq.png)

------

#### 10、QOS

 Site3中AR20上一个用户（ loopback0模拟）和AR18的一个用户（loopback0模拟）要进行语音通信，每路语音需要64Kbps的带宽，目前从AR20到AR18的语音质量不够好，需要在AR19上部署QOS，以保证语音流量的服务质量；<font color='red'>语音流量基于UDP目标端口号详见题本</font>

##### 解决方案

###### 一、R19 配置入向复杂流分类将语音流量进行重标记                                                                                                                                                                                                                                                                                                                                                                                            

1. 定义 ACL

查看 acl

```sql
disp acl all 
  Total quantity of nonempty ACL number is 1 
  Advanced ACL UDP 3999, 1 rule
  Acl's step is 5
   rule 2 deny ip (10607 matches)
```

```sql
acl name UDP 3999
	rule 2 deny ip   # 务必删除
```

加上 rule 1，用来匹配去往 R18 的语音流量

```sql
 rule 1 permit udp source 10.3.1.20 0 destination 10.3.1.18 0 destination-port range 16384 32767 
```

注意：不然会影响 ospf 邻居等等问题

![image-20211019160149188](https://i.loli.net/2021/10/19/xeV5j384SLlIaXP.png)

```sql
	rule 2 deny ip   # 该规则会导致所有其他流量被丢弃，务必删除，哪怕没考察到 QoS，该错误也 务必删除
	undo rule 2
```

2. 定义流分类

```sql
traffic classifier Match-UDP
	if-match acl UDP
```

3. 定义流行为

```sql
traffic behavior Remark-EF
	remark dscp ef
```

4. 定义流策略

```sql
traffic policy Remark-EF
	classifier Match-UDP behavior Remark-EF
```

5. R19 接口入向调用流策略

```sql
interface GigabitEthernet0/0/0
	undo traffic-policy outbound
	traffic-policy Remark-EF inbound
```

------

###### 二、R19配置出向CBQ基于类队列进行优先转发（保证带宽）

1. 定义流分类

```sql
traffic classifier Match-EF
	if-match dscp ef 
```

2. 定义流行为

```sql
traffic behavior CBQ
	queue llq bandwidth 64 cbs 1600
```

3. 定义流策略

```sql
traffic policy CBQ
	classifier Match-EF behavior CBQ
```

4. 接口下出向调用流策略 `CBQ`

```sql
interface Serial3/0/0
	traffic-policy CBQ outbound
```

------

##### 验证

`disp traffic policy user-defined`: 查看用户自定义策略

![image-20211019161858513](https://i.loli.net/2021/10/19/WhpYy9tbZas5Q2I.png)

`disp traffic-policy applied-record`： 查看策略调用结果

![image-20211019162158999](https://i.loli.net/2021/10/19/49KEACjzTbdtMNF.png)

------

#### 11、Sham-link

 Site2与Site3为同一个VPN客户的两个站点，现在AR10与AR20上面的客户（loopback0模拟）都能互通；请解决此问题；并且要求当  AS100连接正常的时候，两个客户的数据包通信必须经过AS100；但是AS100出现问题的时候，两个站点可以通过备份链路进行通信；

##### 一、解决 MPLS-VPN 单域问题

1.  控制层面故障

   A. PE 设备是否做了路由的引入 --- 发现都有

   R6 配置

   ```sql
   ospf 110 router-id 10.3.128.6 vpn-instance 2
   	import-route bgp
   ```

   ```sql
   bgp 100
     ipv4-family vpn-instance 2 
       import-route ospf 110
   ```

   ------

   B. PE 借助 RR(R7) 传递路由时，对等体建立错误 --- 无错误

   ```sql
   <AR7>disp bgp vpnv4 all peer 
     Peer            V          AS  MsgRcvd  MsgSent  OutQ  Up/Down       State PrefRcv
     100.1.1.1       4         100     1022     1114     0 16:51:49 Established    5
     100.1.1.2       4         100     1875     1954     0 0029h59m Established    7
     100.1.1.6       4         100     1829     1849     0 0029h43m Established    21
     100.1.1.13      4         100     1079     1042     0 16:51:27 Established    21
   ```

   ------

   C. RR 配置错误，没有指定特定的客户端 --- R6、R13 没指定客户端

   D. RR 没有关闭 `RT` 值检测  --- 在做 option b 时已关

   R7 配置

   ```sql
   bgp 100
     ipv4-family vpnv4
       undo policy vpn-target
       peer 100.1.1.1 enable
       peer 100.1.1.1 reflect-client
       peer 100.1.1.2 enable
       peer 100.1.1.2 reflect-client
       peer 100.1.1.6 enable
       peer 100.1.1.6 reflect-client
       peer 100.1.1.13 enable
       peer 100.1.1.13 reflect-client
   ```

   这里可以在 `R13` 上查看 vpnv4 路由了 --- 如果没有就要检查下 RT 值了

   `disp bgp vpnv4 all routing-table`

   `disp ip rou vpn-instance 2`

   ```sql
    VPN-Instance 2, Router ID 100.1.1.3:
   
    Total Number of Routes: 28
         Network            NextHop        MED        LocPrf    PrefVal Path/Ogn
    *>   10.2.1.10/32       0.0.0.0         100                   0      ?
    * i                     100.1.1.6       3          100        0      ?
    *>   10.2.1.11/32       0.0.0.0         99                    0      ?
    * i  
   ```

   ------

   E. PE 两端 `RT` 值配置错误

   R6、R13

   ```sq;
   ip vpn-instance 2
   disp this
   ```

   ------

2. 转发层面故障

   A. 底层 IGP 故障

   ```sql
   disp ospf peer brief
   ```

   B. 底层 LDP 故障 --- 接口 mpls 未开

   ```sql
   disp mpls ldp session all
   ```

------

##### 二、配置 sham-link

1. R6 和 R13 之间建立 sham-link

   A. 使用 LoopBack2 作为 sham-link 源目地址

   ​	OSPF 私网进程的下 `area 1` 配置 sham-link

   R6 配置

   ```sql
   ospf 110 router-id 10.3.128.6 vpn-instance 2
     import-route bgp
     area 0.0.0.0 
       sham-link 100.1.136.13 100.1.136.6
   ```

   ​	发现 sham-link 区域错误以及源目地址错误

   ```sql
   area 0.0.0.1
   	sham-link 100.1.136.6 100.1.136.13
   ```

   R13 配置

   ```sql
   area 0.0.0.1 
     sham-link 100.1.136.6 100.1.136.13
   ```

   ​	发现 sham-link 源目地址错误 

   ```sql
   area 0.0.0.1   
     sham-link 100.1.136.13 100.1.136.6
   ```

   ------

   B. 该接口务必绑定进 `VPN-instance 2`

   R6 配置

   ```sql
   [AR6]disp ip vpn-instance 2 int
    VPN-Instance Name and ID : 2, 1
     Interface Number : 2 
     Interface list : GigabitEthernet0/0/2,  # 有对 ospf 私网的接口
                      LoopBack2 				# 有 loop2
   ```

   R13 配置 --- 可能考场是 `LoopBack2` 未宣告	

   ```sql
   [AR13]disp ip vpn-instance 2 interface
    VPN-Instance Name and ID : 2, 1
     Interface Number : 3 
     Interface list : GigabitEthernet0/0/0, # 有对 bgp 私网的接口
                      LoopBack1,  
                      LoopBack2  # 发现也有
   ```

   ------

   C. 该接口地址必须通过 bgp 私网实例进行通告

   R6 配置 --- 发现有

   ```sql
   bgp 100
     ipv4-family vpn-instance 2 
       network 100.1.136.6 255.255.255.255 
   ```

   R13 配置 --- <font color='red'>未宣告</font>

   ```sql
   bgp 100
     ipv4-family vpn-instance 2 
       network 100.1.136.13 255.255.255.255 
   ```

   ------

   D. 该接口`不能` 宣告进 ospf 私网当中 --- 会导致 sham-link 起不来

   R6 配置

   ```sql
   ospf 110 router-id 10.3.128.6 vpn-instance 2
    import-route bgp
    area 0.0.0.1 
     network 10.2.1.6 0.0.0.0 
     network 10.2.128.9 0.0.0.0 
     network 100.1.136.6 0.0.0.0   # 发现有将自己的 LoopBack2 宣告进来
     network 100.1.136.13 0.0.0.0  # 宣告 R13 的 LoopBack2 干嘛？ 要删掉
     sham-link 100.1.136.6 100.1.136.13
     
   ```
   
   改正
   
   ```sql
   ospf 110
   	area 1
       undo network 100.1.136.6 0.0.0.0 
       undo network 100.1.136.13 0.0.0.0 
   ```
   
   R13 配置
   
   ```sql
   ospf 110 router-id 10.3.128.13 vpn-instance 2
    import-route bgp
    area 0.0.0.1 
     network 10.3.1.13 0.0.0.0 
     network 10.3.128.13 0.0.0.0 
     network 100.1.136.13 0.0.0.0  # 发现有将自己的 LoopBack2 宣告进来
     sham-link 100.1.136.13 100.1.136.6
   ```
   
   改正
   
   ```sql
   ospf 110
   	area 1
       undo network 100.1.136.13 0.0.0.0 
   ```
   
   验证 sham-link
   
   R6
   
   ![image-20211019180942124](https://i.loli.net/2021/10/19/E6gORvLBdQioFX8.png)
   
   R13
   
   ![image-20211019181008469](https://i.loli.net/2021/10/19/okTj7PBxvmNlnKC.png)

------

##### 三、调整开销值

![image-20211019181504735](https://i.loli.net/2021/10/19/O7mL8cJYUwzrjaN.png)

如图所示

R20 --- R10 

1. 从 `mpls 骨干网` 的 cost = 1 + 48 + 1 + 1 + 1 + 1 = 53
2. 从 备份链路的 cost  = 48 + 1 = 49

所以我们 R11、R20 的 链路开销为 100

R11 配置

```sql
interface Serial3/0/1
	ospf cost 100
```

R20 配置

```sql
interface Serial3/0/0
	ospf cost 100
```

------

##### 验证：

R10 --- R20

```sql
[AR10]ping -r -a 10.2.1.10 10.3.1.20
  PING 10.3.1.20: 56  data bytes, press CTRL_C to break
    Reply from 10.3.1.20: bytes=56 Sequence=1 ttl=251 time=130 ms
      Record Route: 
      10.2.128.6
      10.2.128.9
      10.3.128.13
      10.3.159.15
      10.3.192.19
      10.3.1.20
      10.3.159.19
      10.3.128.15
      10.3.128.13
```

哎，怎么回来的时候到 13 就没有了呢？--- 后面百度知道 **由于 ping -r 命令最多只能跟踪到9个路由，多以后面的不予显示**， 后来查文档， -r 后面没有<参数> 可跟

接着分段验证

我先 tracert R10 --- R20，看看走的是哪个网络 --- 发现走的是 `mpls 骨干网`

```sql
<AR10>tracert  -a 10.2.1.10 10.3.1.20
 traceroute to  10.3.1.20(10.3.1.20), max hops: 30 ,packet length: 40,press CTRL
_C to break 
 1 10.2.106.6 30 ms  10 ms  20 ms 
 2 10.2.128.9 10 ms  10 ms  10 ms 
 3 100.1.67.7 40 ms  30 ms  40 ms 
 4 100.1.78.8 40 ms  40 ms  40 ms 
 5 10.3.128.13 50 ms  40 ms  40 ms 
 6 10.3.128.15 50 ms  80 ms  60 ms 
 7 10.3.159.19 70 ms  60 ms  70 ms 
 8 10.3.192.20 50 ms  90 ms  60 ms

```

再 tracert R20 --- R10， 看看回去走的哪个网络 --- 发现走的是 `mpls 骨干网`

```sql
[AR20]tracert -a 10.3.1.20 10.2.1.10
 traceroute to  10.2.1.10(10.2.1.10), max hops: 30 ,packet length: 40,press CTRL
_C to break 
 1 10.3.192.19 20 ms  20 ms  20 ms 
 2 10.3.159.15 30 ms  40 ms  30 ms 
 3 10.3.128.13 50 ms  60 ms  70 ms 
 4 100.1.138.8 80 ms  80 ms  50 ms 
 5 100.1.78.7 90 ms  70 ms  80 ms 
 6 10.2.128.9 60 ms  90 ms  60 ms 
 7 10.2.128.6 60 ms  80 ms  70 ms 
 8 10.2.106.10 80 ms  70 ms  70 ms
```

------

```sql
rule 5 permit source 10.0.1.10 0.0.0.0
rule 5 permit source 10.0.1.10 0.0.0.255 
```

上面两条命令的区别？

------

#### 12、telnet

 Site3中，AR16 AR17 AR18帧中继网络中运行ospf，使用默认的网络类型；要求AR18能通过telnet远程管理AR16、AR17；现在AR18 无法远程管理；解决此问题并满足以下条件：

+ 要求AR16的telnet认证方式为AAA，AR16上存在两个用户：admin用户级别为15级，guest用户级别为1级，要求两个用户都能通过telnet 登录； 

+ 要求AR17的认证方法为password，支持命令要求与截图一致---------------考场具体看图，我不贴了

##### 解决方案

###### 	一、R16的 telnet 认证方式为 AAA，admin 用户级别为 15 级，guest 用户级别为 1 级

1. 先去看下 aaa

```sql
aaa 
  local-user admin password cipher hcie
  local-user admin privilege level 15
  local-user admin service-type telnet
  local-user guest password cipher hcie
  local-user guest privilege level 1
  local-user guest service-type telnet
```

2. 再去看下 vty --- 改下认证方式为 aaa

```sql
user-interface vty 0 4
	authentication-mode aaa
```

3. 注意：考场可能会加上以下错误

   + vty 下只放行了 ssh，并未放行 telnet

   ```sql
   user-interface vty 0 4
   	protocol inbound ssh
   ```

   直接改成 telnet 就好了

   ```sql
   user-interface vty 0 4
   	protocol inbound telnet
   ```

   + 可能存在流量过滤

   ```sql
   acl number 2000  
   	rule 5 deny 
   user-interface vty 0 4
   	acl 2000 inbound
   ```

   不删除 acl，加上业务网段规则就好了

   ```sql
   rule 1 permit source 10.3.0.0 0.0.255.255
   ```

------

######   二、R17 的认证方法为 password

```sql
user-interface vty 0 4
	authentication-mode password
	set authentication password cipher hcie  # 修改
	user privilege level 0  # 加上
```

------

##### 验证

看考场图

![image-20220306144821811](https://s2.loli.net/2022/03/06/9FGQxucsNhW3lvC.png)

![image-20220306145044819](https://s2.loli.net/2022/03/06/5BoivNwGLm6xYHa.png)

![image-20220306145137286](https://s2.loli.net/2022/03/06/HOSQXwxARjln8kM.png)

------

#### 13、IPV6

 Site2与Site3配置了IPV6，并且运行OSPFV3协议；参与的设备有AR10、AR11、AR18、AR20；AR18与AR20之间通过tunnel相通；
 现在环境中的IPV6 CLIENT 13、IPV6 CLIENT 9 、IPV6 CLIENT10无法实现互相通信，请解决；

![image-20211020002820417](https://i.loli.net/2021/10/20/Z48TmRgIcaJlsNu.png)

##### 解决方案

###### 一、查看 tunnel，GRE 隧道是否配置错误

1. R20 配置

```sql
interface Tunnel0/0/100
  description 10.3.1.18
  ipv6 enable 
  ipv6 address 2002:100:100::20/64 
  ipv6 address FE80::20 link-local
  ospfv3 1 area 0.0.0.0
  tunnel-protocol gre
  source LoopBack0
  destination 10.3.1.18
  gre key 122   # 这里有误
```

改正

```sql
interface Tunnel0/0/100
	gre key 123
```

1. R18 配置 --- 没问题

```sql
interface Tunnel0/0/100
  description 10.3.1.20
  ipv6 enable 
  ipv6 address 2002:100:100::18/64 
  ipv6 address FE80::18 link-local
  ospfv3 1 area 0.0.0.1
  tunnel-protocol gre
  source LoopBack0
  destination 10.3.1.20
  gre key 123
```

------

###### 二、查看 ospfv3 配置

1. 检查 ospfv3 邻居关系

   ```sql
   disp ospfv3 peer
   ```

   发现 R20 --- R11 邻居关系没起来

   发现 R20 --- R18 邻居关系没起来

2. R20 配置

   ```sql
   ospfv3 1
     router-id 10.3.1.20
     area 0.0.0.0
       abr-summary 2002:10:3:209:: 64 not-advertise
   ```

   看见有个汇总，一看是 client9 的，他是不通告，相当于过滤

   ```sql
   ospfv3 1
     area 0.0.0.0
       abr-summary 2002:10:3:209:: 64
   ```

3. R11 配置

   ```sql
   ospfv3 1
   	router-id 10.3.1.20
   ```

   发现 `route-id` 冲突

   ```sql
   ospfv3 1
   	undo router-id
   	router-id 10.3.1.11
   ```

4. 检查 ospfv3 邻居关系 --- 发现 R11 已经和 R10，R20 建立好了

   ```sql
   [AR11]disp ospfv3 peer 
   OSPFv3 Process (1)
   OSPFv3 Area (0.0.0.1)
   Neighbor ID     Pri  State            Dead Time Interface            Instance ID
   10.2.1.10         1  Full/DR          00:00:34  GE0/0/0                        0
   10.3.1.20         1  Full/-           00:00:34  S3/0/1 
   ```

5. R18 配置

   发现 tunnel0/0/100 中 ospfv3 区域配错了

   ```sql
   int t0/0/100
   	 ospfv3 1 area 0.0.0.0
   ```

6. R20 配置错误静默接口 

   ```sql
   ospfv3 1 
   	silent-interface Tunnel0/0/100
   ```

   改正

   ```sql
   ospfv3 1 
   	undo silent-interface Tunnel0/0/100	
   ```

7. 查看 R20 ospfv3 邻居关系 --- 没问题了

   ```sql
   [AR20]disp ospfv3 peer 
   OSPFv3 Process (1)
   OSPFv3 Area (0.0.0.0)
   Neighbor ID     Pri  State            Dead Time Interface            Instance ID
   10.3.1.18         1  Full/-           00:00:35  Tun0/0/100                     0
   OSPFv3 Area (0.0.0.1)
   Neighbor ID     Pri  State            Dead Time Interface            Instance ID
   10.3.1.11         1  Full/-           00:00:30  S3/0/0                         0
   ```

------

##### 验证

所有 IPv6 客户端验证互访即可，如果存在无法互访检查连接客户端对应接口 OSPFv3 是否启用 

**IPV6 CLIENT13、IPV6 CLIENT9 、IPV6 CLIENT10 互访成功**

<font color='red'>注意：VRRP6 需求中新增 Clinet14 可能无法通信成功，VRRP6 在 ENSP 中有 bug</font>

------

#### 14、BGP IPV4互访

 要求AS100中AR12的loopback0口能够访问AS200中的AR9的loopback0口；但是现在两个loopback0口地址无法访问；请解决该问题

![image-20211020002955080](https://i.loli.net/2021/10/20/i9nzmSkWOMJsu2h.png)

##### 可能故障原因

1. 对等体没有建立好
2. RR 客户端没有指定
3. ASBR 上未把 IGP 引入进 BGP
4. 各环回口没有宣告进 IGP
5. AR2、AR4、AR5 需要配置下一跳本地

注意：

```sql
bgp 100
	bgp # 错误输入
	# 则会添加如下命令
	# 导致路由只存在于BGP路由表，不加入IP路由表
	bgp 100 bgp-rib-only 
```

------

##### 解决方案

###### 一、检查 bgp 邻居

1. R12 配置 --- 发现没有与 R7 建立邻居

   进入 bgp 100 查看 没有用 更新源 LoopBack0 

```sql
bgp 100
	router-id 100.1.1.12
  peer 100.1.1.7 as-number 100 
  peer 100.1.1.7 connect-interface LoopBack0
```

2. R7 配置 --- 发现对 R12 的 as 号错误，删掉配置，并且未配置客户端

```sql
 bgp 100
   peer 100.1.1.12 as-number 200 
   peer 100.1.1.12 connect-interface LoopBack0
   peer 100.1.1.12 reflect-client
   # 注意该条命令不是错误配置，题本要求 R13 不要承载 IPv4 单播路由，只传递 VPNv4
   undo peer 100.1.1.13 enable	
```

###### 二、检查路由发布

1. AS200 在 R4 和 R5 上将 IS-IS 引入进 BGP

```sql
bgp 200
  ipv4-family unicast
    import-route isis 200
```

2. AS100 在 R2 上将 OSPF 引入进 BGP

```sql
bgp 100
  ipv4-family unicast
    import-route ospf 100 route-policy ospf_bgp
```

发现有一个 route-policy，查看一下

```sql
[AR2-bgp]disp route-policy ospf_bgp 
Route-policy : ospf_bgp
  permit : 1 (matched counts: 164)
    Match clauses : 
      if-match acl 2000
```

接着查看 acl 2000 --- 发现就是 匹配环回口的

```sql
acl 2000  
  rule 5 permit source 100.1.1.8 0 
  rule 10 permit source 100.1.1.21 0 
  rule 15 permit source 100.1.1.22 0 
  rule 30 permit source 100.1.1.2 0 
  rule 1000 deny 
```

加上 

```sql
  rule 1 permit source 100.1.1.0 0.0.0.255 
```

3. 逐个检查 路由

   R12 --- R7 --- R2 --- R4 --- R5 --- R9

   ```sql
   [AR2]disp bgp routing-table  # 发现自己 AS 环回口都有了，对面的也有了
   
    BGP Local router ID is 100.1.1.2 
    Status codes: * - valid, > - best, d - damped,
                  h - history,  i - internal, s - suppressed, S - Stale
                  Origin : i - IGP, e - EGP, ? - incomplete
                  
    Total Number of Routes: 30
         Network            NextHop        MED        LocPrf    PrefVal Path/Ogn
   
    *>   100.1.1.1/32       0.0.0.0         1                     0      ?
    *>   100.1.1.2/32       0.0.0.0         0                     0      ?
    *>   100.1.1.3/32       0.0.0.0         1                     0      ?
    *>   100.1.1.6/32       0.0.0.0         2                     0      ?
    *>   100.1.1.7/32       0.0.0.0         1                     0      ?
    *>   100.1.1.8/32       0.0.0.0         2                     0      ?
    *>   100.1.1.12/32      0.0.0.0         3                     0      ?
    *>   100.1.1.13/32      0.0.0.0         2                     0      ?
    *>   100.1.1.21/32      0.0.0.0         2                     0      ?
    *>   100.1.1.22/32      0.0.0.0         3                     0      ?
    *>   200.1.1.4/32       200.100.24.4    0                     0      200?
    *                       200.100.25.5    10                    0      200?
    *>   200.1.1.5/32       200.100.25.5    0                     0      200?
    *                       200.100.24.4    10                    0      200?
    *>   200.1.1.9/32       200.100.24.4    10                    0      200?
    *                       200.100.25.5    10                    0      200?
    *>   200.1.1.23/32      200.100.24.4    20                    0      200?
    *                       200.100.25.5    20                    0      200?
    *>   200.1.1.99/32      200.100.24.4    10                    0      200?
    *                       200.100.25.5    10                    0      200?
    *>   200.1.29.0         200.100.24.4    30                    0      200?
    *                       200.100.25.5    30                    0      200?
    *>   200.1.45.0         200.100.24.4    0                     0      200?
    *                       200.100.25.5    0                     0      200?
    *>   200.1.49.0         200.100.24.4    0                     0      200?
    *                       200.100.25.5    20                    0      200?
    *>   200.1.59.0         200.100.25.5    0                     0      200?
    *                       200.100.24.4    20                    0      200?
    *>   200.1.239.0        200.100.24.4    20                    0      200?
    *                       200.100.25.5    20                    0      200?
   ```

   ```sql
   [AR9]disp bgp routing-table # 发现自己 AS 环回口都有了，对面的也有了
   
    BGP Local router ID is 200.1.1.9 
    Status codes: * - valid, > - best, d - damped,
                  h - history,  i - internal, s - suppressed, S - Stale
                  Origin : i - IGP, e - EGP, ? - incomplete
   
    Total Number of Routes: 40
         Network            NextHop        MED        LocPrf    PrefVal Path/Ogn
   
    *>i  100.1.1.1/32       200.1.1.4       1          100        0      100?
    * i                     200.1.1.5       1          100        0      100?
    *>i  100.1.1.2/32       200.1.1.4       0          100        0      100?
    * i                     200.1.1.5       0          100        0      100?
    *>i  100.1.1.3/32       200.1.1.4       1          100        0      100?
    * i                     200.1.1.5       1          100        0      100?
    *>i  100.1.1.6/32       200.1.1.4       2          100        0      100?
    * i                     200.1.1.5       2          100        0      100?
    *>i  100.1.1.7/32       200.1.1.4       1          100        0      100?
    * i                     200.1.1.5       1          100        0      100?
    *>i  100.1.1.8/32       200.1.1.4       2          100        0      100?
    * i                     200.1.1.5       2          100        0      100?
    *>i  100.1.1.12/32      200.1.1.4       3          100        0      100?
    * i                     200.1.1.5       3          100        0      100?
    *>i  100.1.1.13/32      200.1.1.4       2          100        0      100?
    * i                     200.1.1.5       2          100        0      100?
    *>i  100.1.1.21/32      200.1.1.4       2          100        0      100?
    * i                     200.1.1.5       2          100        0      100?
    *>i  100.1.1.22/32      200.1.1.4       3          100        0      100?
    * i                     200.1.1.5       3          100        0      100?
    *>i  200.1.1.4/32       200.1.1.5       10         100        0      ?
      i                     200.1.1.4       0          100        0      ?
    *>i  200.1.1.5/32       200.1.1.4       10         100        0      ?
      i                     200.1.1.5       0          100        0      ?
    *>i  200.1.1.9/32       200.1.1.4       10         100        0      ?
    * i                     200.1.1.5       10         100        0      ?
    *>i  200.1.1.23/32      200.1.1.4       20         100        0      ?
    * i                     200.1.1.5       20         100        0      ?
    *>i  200.1.1.99/32      200.1.1.4       10         100        0      ?
    * i                     200.1.1.5       10         100        0      ?
    *>i  200.1.29.0         200.1.1.4       30         100        0      ?
    * i                     200.1.1.5       30         100        0      ?
    *>i  200.1.45.0         200.1.1.4       0          100        0      ?
    * i                     200.1.1.5       0          100        0      ?
    *>i  200.1.49.0         200.1.1.4       0          100        0      ?
    * i                     200.1.1.5       20         100        0      ?
    *>i  200.1.59.0         200.1.1.5       0          100        0      ?
    * i                     200.1.1.4       20         100        0      ?
    *>i  200.1.239.0        200.1.1.4       20         100        0      ?
    * i                     200.1.1.5       20         100        0      ?
   ```

------

###### 三、配置下一跳本地

1. AR2 配置

```sql
bgp 100
 peer 100.1.1.7 next-hop-local 
```

2. AR4、AR5 配置

```sql
bgp 200
 peer 200.1.1.9 next-hop-local 
```

------

##### 验证

```sql
<AR12>ping -a 100.1.1.12 200.1.1.9
  PING 200.1.1.9: 56  data bytes, press CTRL_C to break
    Reply from 200.1.1.9: bytes=56 Sequence=1 ttl=251 time=70 ms
    Reply from 200.1.1.9: bytes=56 Sequence=2 ttl=251 time=50 ms
    Reply from 200.1.1.9: bytes=56 Sequence=3 ttl=251 time=50 ms
    Reply from 200.1.1.9: bytes=56 Sequence=4 ttl=251 time=50 ms
    Reply from 200.1.1.9: bytes=56 Sequence=5 ttl=251 time=40 ms
```

------



#### 15、BGP IPV6互访

 AS200中AR9、AR4、AR5上面各有一个IPV6客户站点（loopback1模拟），AS100中有AR2 、R7上各有一个IPV6客户站点（loopback1模 拟）， 要求这几个站点都能互通，并且要求双向路径要优选AR2-AR4之间的链路，一旦主用链路出现问题在使用备用链路AR2-AR5；现在这几个站点之间的互通存在部分问题，请解决；

<font color='red'>注意：考场上选路策略只允许在 AS100 中配置</font>

##### 分析

站在 R2 视角分析进出路由，使用的策略

1. R2 在发出路由的时候将 `med` 值改大或改小，考场上用的是增加 `as-path` 路径
   + 对于发给 R4 的 `as-path` 不做调整
   + 对于发给 R5 的 `as-path` 加长
2. R2 在接收路由的时候将 `LocalPreference` 改大或改小
   + 对于 R4 发来的路由 `LocalPreference` 改大
   + 对于 R5 发来的路由 `LocalPreference` 不做调整

------

##### 可能故障原因

1. BGP ipv6 邻居关系 down
2. BGP 路由发布错误和传递错误
3. BGP ipv6 反射器配置
4. R2 检查对于 R4 的 `LP` 路由策略错误
   + 方向错误
5. R2 检查对于 R5 的 `AS` 路由策略错误
   + AS 号更改为 200 的系统号 --- 导致 AS200 不接受
   + 方向错误

------

##### 解决方案

1. 检查 BGP ipv6 邻居关系

   R2、R9  --- 发现没有问题

   R2 - {R7, R4, R5}, R9 - {R4, R5} 建立邻居关系

   ```sql
   disp bgp ipv6 peer
   ```

2. 检查路由发布 --- 每台路由器通过network命令发布自身的环回口1接口地址

   > 检查自己自治系统的路由发布

   AS 100 内

   R7 查看 --- 发现并没有发布自己的 `LoopBack1`

   ```sql
   ipv6-family unicast
     peer 2002:100:1:1::2 enable
   ```

   加上

   ```sql
   ipv6-family unicast
   	network 2002:100:7:1::1 128 
   ```

   R2 查看 --- 发现有

   ```sql
   ipv6-family unicast
     network 2002:100:2:1::1 128 
   ```

   ------

   AS 200 内 --- 都有

   R9 查看

   ```sql
   ipv6-family unicast
     network 2002:200:9:1::1 128 
   ```

   R4 查看

   ```sql
   ipv6-family unicast
     network 2002:200:4:1::1 128 
   ```

   R5 查看

   ```sql
   ipv6-family unicast
     network 2002:200:5:1::1 128 
   ```

   ------

   > AR2、AR4、AR5 配置下一跳本地

   AR2 配置

   ```sql
   bgp 100
     ipv6-family unicast
       peer 2002:100:1:1::7 next-hop-local
   ```

   AR4 和 AR5 配置 

   ```sql
   bgp 200
     ipv6-family unicast
       peer 2002:200:1:1::9 next-hop-local
   ```

3. 检查 ipv6 RR 配置

   + R9 - R4, R5 --- <font color='red'>没有指定</font>

   ```sql
   bgp 200
     ipv6-family unicast
       peer 2002:200:1:1::4 enable
       peer 2002:200:1:1::4 reflect-client
       peer 2002:200:1:1::5 enable
       peer 2002:200:1:1::5 reflect-client
   ```

   + R7 - R2 配不配置无所谓，因为 E - I
   + 此时 R5 有两个下一跳了，一个是 R2，一个是 R4

   ------

   + 选路问题

   R2 配置

   ```sql
   ipv6-family unicast
     peer 2002:100:1:1::7 enable
     peer 2002:100:1:1::7 next-hop-local 
     peer 2002:200:100:24::4 enable
     peer 2002:200:100:24::4 route-policy LP import
     peer 2002:200:100:24::4 route-policy LP export
     peer 2002:200:100:25::5 enable
     peer 2002:200:100:25::5 route-policy AS-Path import
   ```

   > 先看对于 R4 的 `LP` 策略

   明显错误，只能 `import` 方向有效

   ```sql
   undo peer 2002:200:100:24::4 route-policy LP export
   ```

   不放心，在查看以下 `LP`，没问题是改大 LP

   ```sql
   disp route-policy LP
   Route-policy : LP
     permit : 1 (matched counts: 0)
       Apply clauses : 
         apply local-preference 200
   ```

   > 看对于 R5 的 `AS-Path` 策略

   明显错误，只能 `export` 方向有效

   ```sql
   undo peer 2002:200:100:25::5 route-policy AS-Path import
   peer 2002:200:100:25::5 route-policy AS-Path export
   ```

   不放心，在查看以下 `AS-Path`， 怎么添加 AS 号有 200？那么对面 R5 拿到了不就拒收了吗？

   ```sql
   disp route-policy AS-Path 
   Route-policy : AS-Path
     permit : 1 (matched counts: 0)
       Apply clauses : 
         apply as-path 200 200 additive
   ```

   ```sql
   route-policy AS-Path permit node 1 
   	apply as-path 100 100 additive
   ```

   > 查看 bgp ipv6 路由表

   R2 查看 --- 发现 R4、R5、R9 走的都是 `2-4`

   ```sql
   [AR2]disp bgp ipv6 routing-table 
    BGP Local router ID is 100.1.1.2 
    Status codes: * - valid, > - best, d - damped,
                  h - history,  i - internal, s - suppressed, S - Stale
                  Origin : i - IGP, e - EGP, ? - incomplete
   
    Total Number of Routes: 8
    *>  Network  : 2002:100:2:1::1                          PrefixLen : 128    
        NextHop  : ::                                       LocPrf    :          
        MED      : 0                                        PrefVal   : 0         
        Label    : 
        Path/Ogn : i
    *>i Network  : 2002:100:7:1::1                          PrefixLen : 128    
        NextHop  : 2002:100:1:1::7                          LocPrf    : 100       
        MED      : 0                                        PrefVal   : 0         
        Label    : 
        Path/Ogn : i
    *>  Network  : 2002:200:4:1::1                          PrefixLen : 128   ### 此处 
        NextHop  : 2002:200:100:24::4                       LocPrf    : 200       
        MED      : 0                                        PrefVal   : 0         
        Label    : 
        Path/Ogn : 200  i
    *                                            
        NextHop  : 2002:200:100:25::5                       LocPrf    :          
        MED      :                                          PrefVal   : 0         
        Label    : 
        Path/Ogn : 200  i
    *>  Network  : 2002:200:5:1::1                          PrefixLen : 128    ### 此处
        NextHop  : 2002:200:100:24::4                       LocPrf    : 200       
        MED      :                                          PrefVal   : 0         
        Label    : 
        Path/Ogn : 200  i
    *                                            
        NextHop  : 2002:200:100:25::5                       LocPrf    :          
        MED      : 0                                        PrefVal   : 0         
        Label    : 
        Path/Ogn : 200  i
    *>  Network  : 2002:200:9:1::1                          PrefixLen : 128   ### 此处 
        NextHop  : 2002:200:100:24::4                       LocPrf    : 200       
        MED      :                                          PrefVal   : 0         
        Label    : 
        Path/Ogn : 200  i
    *                                            
        NextHop  : 2002:200:100:25::5                       LocPrf    :          
        MED      :                                          PrefVal   : 0         
        Label    : 
        Path/Ogn : 200  i
   ```

   R4 查看 --- 发现去 `R2、R7` 都是走的 R2

   ```sql
   <AR4>disp bgp ipv6 routing-table 
   
    BGP Local router ID is 200.1.1.4 
    Status codes: * - valid, > - best, d - damped,
                  h - history,  i - internal, s - suppressed, S - Stale
                  Origin : i - IGP, e - EGP, ? - incomplete
   
    Total Number of Routes: 5
    *>  Network  : 2002:100:2:1::1                          PrefixLen : 128    
        NextHop  : 2002:200:100:24::2                       LocPrf    :     ### 此处     
        MED      : 0                                        PrefVal   : 0         
        Label    : 
        Path/Ogn : 100  i
    *>  Network  : 2002:100:7:1::1                          PrefixLen : 128    
        NextHop  : 2002:200:100:24::2                       LocPrf    :      ### 此处     
        MED      :                                          PrefVal   : 0         
        Label    : 
        Path/Ogn : 100  i
    *>  Network  : 2002:200:4:1::1                          PrefixLen : 128    
        NextHop  : ::                                       LocPrf    :          
        MED      : 0                                        PrefVal   : 0         
        Label    : 
        Path/Ogn : i
    *>i Network  : 2002:200:5:1::1                          PrefixLen : 128    
        NextHop  : 2002:200:1:1::5                          LocPrf    : 100       
        MED      : 0                                        PrefVal   : 0         
        Label    : 
        Path/Ogn : i
    *>i Network  : 2002:200:9:1::1                          PrefixLen : 128    
        NextHop  : 2002:200:1:1::9                          LocPrf    : 100       
        MED      : 0                                        PrefVal   : 0         
        Label    : 
        Path/Ogn : i
   ```

   R5 查看 --- 发现都是走的 R4 --- 那就没问题了

   ```sql
   <AR5>disp bgp ipv6 routing-table 
   
    BGP Local router ID is 200.1.1.5 
    Status codes: * - valid, > - best, d - damped,
                  h - history,  i - internal, s - suppressed, S - Stale
                  Origin : i - IGP, e - EGP, ? - incomplete
   
   
    Total Number of Routes: 7
    *>i Network  : 2002:100:2:1::1                          PrefixLen : 128    ### 此处
        NextHop  : 2002:200:1:1::4                          LocPrf    : 100       
        MED      : 0                                        PrefVal   : 0         
        Label    : 
        Path/Ogn : 100  i
    *                                            
        NextHop  : 2002:200:100:25::2                       LocPrf    :          
        MED      : 0                                        PrefVal   : 0         
        Label    : 
        Path/Ogn : 100 100 100  i
    *>i Network  : 2002:100:7:1::1                          PrefixLen : 128   ### 此处 
        NextHop  : 2002:200:1:1::4                          LocPrf    : 100       
        MED      :                                          PrefVal   : 0         
        Label    : 
        Path/Ogn : 100  i
    *                                            
        NextHop  : 2002:200:100:25::2                       LocPrf    :          
        MED      :                                          PrefVal   : 0         
        Label    : 
        Path/Ogn : 100 100 100  i
    *>i Network  : 2002:200:4:1::1                          PrefixLen : 128    
        NextHop  : 2002:200:1:1::4                          LocPrf    : 100       
        MED      : 0                                        PrefVal   : 0         
        Label    : 
        Path/Ogn : i
    *>  Network  : 2002:200:5:1::1                          PrefixLen : 128    
        NextHop  : ::                                       LocPrf    :          
        MED      : 0                                        PrefVal   : 0         
        Label    : 
        Path/Ogn : i
    *>i Network  : 2002:200:9:1::1                          PrefixLen : 128    
        NextHop  : 2002:200:1:1::9                          LocPrf    : 100       
        MED      : 0                                        PrefVal   : 0         
        Label    : 
        Path/Ogn : i
   ```

------

##### 验证

R5 --- R7，走的是 4 - 2 - 7 没毛病

```sql
<AR5>tracert ipv6 -a 2002:200:5:1::1 2002:100:7:1::1

 1 2002:200:1:45::4 20 ms  20 ms  30 ms 
 2 2002:200:100:24::2 30 ms  30 ms  20 ms 
 3 2002:100:7:1::1 40 ms  30 ms  30 ms 
```

回来呢？也是 2 - 4 - 5

```sql
<AR7>tracert ipv6 -a 2002:100:7:1::1 2002:200:5:1::1

 1 2002:100:1:27::2 20 ms  20 ms  20 ms 
 2 2002:200:100:24::4 30 ms  30 ms  20 ms 
 3 2002:200:5:1::1 20 ms  20 ms  40 ms 
```

R9 --- R7 都没问题

```sql
<AR9>tracert ipv6 -a 2002:200:9:1::1 2002:100:7:1::1

 1 2002:200:1:49::4 10 ms  20 ms  10 ms 
 2 2002:200:100:24::2 30 ms  20 ms  30 ms 
 3 2002:100:7:1::1 40 ms  30 ms  30 ms 
```

R7 --- R9

```sql
<AR7>tracert ipv6 -a 2002:100:7:1::1 2002:200:9:1::1

 1 2002:100:1:27::2 20 ms  30 ms  20 ms 
 2 2002:200:100:24::4 20 ms  20 ms  20 ms 
 3 2002:200:9:1::1 30 ms  20 ms  20 ms 
```

> 现在把 R2 --- R4 之间的 pos5/0/0 shutdown 看下

R2

```sql
bgp 100
  ebgp-interface-sensitive # 立刻感知到接口的状态变化，从而对等体关系重建，可能跟 pos 链路有关，10s 一次
interface Pos5/0/0
  shutdown
```

验证下是不是 pos 链路 10s 轮询时间间隔

> **应用场景** --- 文档说明
>
> 轮询时间间隔指的是接口发送keepalive报文的周期。
>
> keepalive报文用于链路状态监测维护，接口如果在5个keepalive周期之后仍然无法收到对端的keepalive报文，它就会认为链路发生故障。

说明默认就是 10 * 5 = 50，就相当于是 ebgp 32s 自动重建了

我按照文档把轮询间隔改为 1s，这样就是 5s 就可以感知到会话 down 了

R2、R4

```sql
int pos5/0/0
	timer hold 1
```

R2

```sql
int pos5/0/0
	shutdown
```

发现就在 5s 内 ppp 链路 down 掉， ebgp down 了

------

R5 --- R7，走的是 2 - 7 没毛病

```sql
<AR5>tracert ipv6 -a 2002:200:5:1::1 2002:100:7:1::1

 1 2002:200:100:25::2 30 ms  20 ms  20 ms 
 2 2002:100:7:1::1 20 ms  20 ms  30 ms 
```

回来呢？走 2 - 5

```sql
<AR7>tracert ipv6 -a 2002:100:7:1::1 2002:200:5:1::1

 1 2002:100:1:27::2 20 ms  10 ms  10 ms 
 2 2002:200:5:1::1 30 ms  30 ms  20 ms
```

------



#### 16、NAT

 Site5中通过安全接入AS,client11现在无法通过网关AR27访问到公网AS100、AS200；
 解决此问题已满足以下表项------------考场具体看图，判断是PAT还是Easy-ip，我不贴了

##### 解决方案

1. ##### PPP CHAP 认证

   R9 配置

   ```sql
   interface Serial3/0/0
   	ppp authentication-mode chap 
   aaa
   	local-user hcie password cipher hcie
   	local-user hcie service-type ppp
   ```

   R27 配置

   ```sql
   interface Serial3/0/0
   	ppp chap user hcie
   	ppp chap password cipher hcie
   ```

   ping R9

   ```sql
   [AR27]ping 200.1.209.9
     PING 200.1.209.9: 56  data bytes, press CTRL_C to break
       Reply from 200.1.209.9: bytes=56 Sequence=1 ttl=255 time=10 ms
       Reply from 200.1.209.9: bytes=56 Sequence=2 ttl=255 time=20 ms
       Reply from 200.1.209.9: bytes=56 Sequence=3 ttl=255 time=30 ms
       Reply from 200.1.209.9: bytes=56 Sequence=4 ttl=255 time=20 ms
       Reply from 200.1.209.9: bytes=56 Sequence=5 ttl=255 time=20 ms
   ```

   

2. NAT 问题

   考场具体看图，判断是PAT还是Easy-ip

   + PAT 是有地址池的
   + Easy-ip 直接使用物理口，pppoe 的话是使用这种

   > PAT 配置 --- R27

   1. 先写一个 acl

   ```sql
   acl 2000  
   	rule 1 permit 
   ```

   2. 地址池

   ```sql
   nat address-group 1 200.1.209.1 200.1.209.5
   ```

   3. 接口出向调用

   ```sql
   interface Serial3/0/0
   	nat outbound 2000 address-group 1 
   ```

   4. 默认路由没有配置 --- 考场会给你配好

   ```sql
   ip route-static 0.0.0.0 0.0.0.0 Serial3/0/0
   																# 这里直接配置出接口 ok 不？
   																# ppp 链路是 ok 的，因为只有两端
   																# MA 只写出接口没有，因为对端设备很多，你不知道发给谁
   																# 而且你怎么进行数据封装呢？没有具体下一跳，无法完成 arp
   ```

   ##### 验证 1：

   ```sql
   PC>ping 200.1.1.9
     Ping 200.1.1.9: 32 data bytes, Press Ctrl_C to break
     From 200.1.1.9: bytes=32 seq=1 ttl=254 time=31 ms
     From 200.1.1.9: bytes=32 seq=2 ttl=254 time=32 ms
     From 200.1.1.9: bytes=32 seq=3 ttl=254 time=15 ms
     From 200.1.1.9: bytes=32 seq=4 ttl=254 time=16 ms
     From 200.1.1.9: bytes=32 seq=5 ttl=254 time=31 ms
   ```

   ![image-20211020154318455](https://i.loli.net/2021/10/20/QF2ZOvRCuLP1I5d.png)

   ------

   > Easy-ip 配置 --- R27

   1. 先写一个 acl

   ```sql
   acl 2000  
   	rule 1 permit 
   ```

   2. 接口出向调用

   ```sql
   interface Serial3/0/0
   	nat outbound 2000 
   ```

   ![image-20211020155642741](https://i.loli.net/2021/10/20/fn2iEr51mHezc9v.png)

   ------

   注意：

   但是 pc  访问 AS200，AS100 都是不通的

   ```sql
   PC>ping 200.1.1.23
   PC>ping 100.1.1.2
   ```

   可以在 AS100 R2 设备上查看有没有 nat 转换过去的路由 `200.1.209.0/24`，并没有

   ```sql
   [AR2]disp ip rou 200.1.209.0
   [AR2]
   ```

   要想有，所以需要 R9 把 `ser3/0/0` 宣告进 isis 200 中

   ```sql
   interface Serial3/0/0
   	isis enable 200
   ```

   R2 查看有没有路由 --- 发现有了

   ```sql
   [AR2]disp ip rou 200.1.209.0
   Route Flags: R - relay, D - download to fib
   ------------------------------------------------------------------------------
   Routing Table : Public
   Summary Count : 1
   Destination/Mask    Proto   Pre  Cost      Flags NextHop         Interface
       200.1.209.0/24  EBGP    255  20          D   200.100.24.4    Pos5/0/0
   ```

   现在 pc 在 ping 一下看看

   ```sql
   PC>ping 200.1.1.23
   
   Ping 200.1.1.23: 32 data bytes, Press Ctrl_C to break
   From 200.1.1.23: bytes=32 seq=1 ttl=253 time=47 ms
   From 200.1.1.23: bytes=32 seq=2 ttl=253 time=15 ms
   From 200.1.1.23: bytes=32 seq=3 ttl=253 time=32 ms
   From 200.1.1.23: bytes=32 seq=4 ttl=253 time=31 ms
   From 200.1.1.23: bytes=32 seq=5 ttl=253 time=15 ms
   
   PC>ping 100.1.1.2
   
   Ping 100.1.1.2: 32 data bytes, Press Ctrl_C to break
   From 100.1.1.2: bytes=32 seq=1 ttl=252 time=15 ms
   From 100.1.1.2: bytes=32 seq=2 ttl=252 time=16 ms
   From 100.1.1.2: bytes=32 seq=3 ttl=252 time=31 ms
   From 100.1.1.2: bytes=32 seq=4 ttl=252 time=47 ms
   From 100.1.1.2: bytes=32 seq=5 ttl=252 time=31 ms
   ```

------

#### final、 TS2.0 完结

​	历经 3 天，终于把这 16 道需求排完了。我将奔赴于 TS3.0 --- 2021 年 10 月 20 日 下午 4 点

![image-20211020155955104](https://i.loli.net/2021/10/20/YpIAkgEoQxrycCO.png)

