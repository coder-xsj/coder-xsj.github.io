## LAB-1







### 1 L2+VRRP（16分）

#### 1.1 链路聚合

假设SW1不支持LACP，SW1和SW2互连的接口需要捆绑成一个二层逻辑接口。逻辑接口成员根据源目的MAC进行负载分担。 

解法：

SW1 、SW2

```sql
interface Eth-Trunk12
	load-balance src-dst-mac
interface GigabitEthernet0/0/23
	eth-trunk 12
interface GigabitEthernet0/0/24
	eth-trunk 12
```

验证：

SW1 、SW2

查看：According to `SA-XOR-DA`，还有接口 `up`

```sql
disp eth-trunk 12
Eth-Trunk12's state information is:
WorkingMode: NORMAL         Hash arithmetic: According to SA-XOR-DA           
Least Active-linknumber: 1  Max Bandwidth-affected-linknumber: 8              
Operate status: up          Number Of Up Port In Trunk: 2                     
--------------------------------------------------------------------------------
PortName                      Status      Weight 
GigabitEthernet0/0/23         Up          1      
GigabitEthernet0/0/24         Up          1 
```

##### 清除接口配置

```sql
clear configuration interface Ethernet 0/0/10
Warning: All configurations of the interface will be cleared, and its state will
 be shutdown. Continue? [Y/N] :y
 ------------------------------------
 undo shutdown
```

#### 1.2 Link-type

1. SW1,SW2,SW3,SW4互连接口的链路类型为Trunk，允许除VLAN1外的所有VLAN通过。（3） 

解法：

以 SW1 为例

```sql
vlan batch 10 20
interface GigabitEthernet0/0/9
	port link-type trunk
	port trunk allow-pass vlan 2 to 4094
	undo port trunk allow-pass vlan 1
	
interface GigabitEthernet0/0/10
	port link-type trunk
	port trunk allow-pass vlan 2 to 4094
	undo port trunk allow-pass vlan 1

interface eth12
	port link-type trunk
	port trunk allow-pass vlan 2 to 4094
	undo port trunk allow-pass vlan 1
```

注意：将SW3和SW4连接PC1、Server1的接口分别划分进对应VLAN

验证：

```sql
[SW1]disp port vlan active 
T=TAG U=UNTAG
-------------------------------------------------------------------------------
Port                Link Type    PVID    VLAN List
-------------------------------------------------------------------------------
Eth-Trunk12         trunk        1       T: 10 20
GE0/0/1             hybrid       1       U: 1
GE0/0/2             hybrid       1       U: 1
GE0/0/3             hybrid       1       U: 1
GE0/0/4             hybrid       1       U: 1
GE0/0/5             hybrid       1       U: 1
GE0/0/6             hybrid       1       U: 1
GE0/0/7             hybrid       1       U: 1
GE0/0/8             hybrid       1       U: 1
GE0/0/9             trunk        1       T: 10 20
GE0/0/10            trunk        1       T: 10 20
```

2. CE1，CE2的VRRP虚拟IP地址10.3.1.254，为PC1的网关。CE1会周期性发送 sender ip 为10.3.1.254，源MAC为00-00-5E-00-01-`01`的免费 ARP。PC1与网关之间的数据包封装在VLAN10中（PC1收发untag帧） 
3. CE1,CE2的VRRP虚拟IP地址10.3.2.254，为Sever1的网关。CE2会周期性发送sender ip为10.3.2.254，源MAC为00-00-5E-00-01-`02`的免费 ARP。Sever1与网关之间的数据包封装在VLAN20中（sever1收发untag帧）
4. VRRP的master设备重启时，在Ge0/0/2变为UP1分钟后，才能重新成为master.（4）

分析：

![image-20211005220714069](https://i.loli.net/2021/10/05/4BzrxLTYyADs6v2.png)

通过 CE1会周期性发送 sender ip 为10.3.1.254，源MAC为00-00-5E-00-01-`01`的免费 ARP

得出 CE1 为 vlan 10 10.3.1.254 master 设备， `vrip` = 1

同理：CE2 为 vlan 20 10.3.2.254 master 设备，`vrid` = 2



解法：

注意：SW1、SW2连接CE1、CE2需要配置为trunk，CE1和CE2配置单臂路由

SW1、SW2

```sql
int g0/0/2
	port link-type trunk
	port trunk allow-pass vlan 2 to 4094
	undo port trunk allow-pass vlan 1
```

##### 1. 配置单臂路由

CE1

```sql
interface GigabitEthernet0/0/2.10
 dot1q termination vid 10
 ip address 10.3.1.1 255.255.255.0 
 arp broadcast enable
interface GigabitEthernet0/0/2.20
 dot1q termination vid 20
 ip address 10.3.2.1 255.255.255.0 
 arp broadcast enable
```

CE2

```sql
interface GigabitEthernet0/0/2.10
 dot1q termination vid 10
 ip address 10.3.1.2 255.255.255.0 
 arp broadcast enable
interface GigabitEthernet0/0/2.20
 dot1q termination vid 20
 ip address 10.3.2.2 255.255.255.0 
 arp broadcast enable
```

##### 2. 配置 MVRRP

CE1

```sql
interface GigabitEthernet0/0/2.10
	vrrp vrid 1 virtual-ip 10.3.1.254
	vrrp vrid 1 priority 120
interface GigabitEthernet0/0/2.20
	vrrp vrid 2 virtual-ip 10.3.2.254
```

CE2

```
interface GigabitEthernet0/0/2.10
	ip address 10.3.1.2 255.255.255.0 
	vrrp vrid 1 virtual-ip 10.3.1.254
interface GigabitEthernet0/0/2.20
	vrrp vrid 2 virtual-ip 10.3.2.254
	vrrp vrid 2 priority 120
```

验证 CE1 、CE2

```
[CE1]disp vrrp  brief 
Total:2     Master:1     Backup:1     Non-active:0      
VRID  State        Interface                Type     Virtual IP     
----------------------------------------------------------------
1     Master       GE0/0/2.10               Normal   10.3.1.254     
2     Backup       GE0/0/2.20               Normal   10.3.2.254 

[CE2]disp vrrp brief 
Total:2     Master:1     Backup:1     Non-active:0      
VRID  State        Interface                Type     Virtual IP     
----------------------------------------------------------------
1     Backup       GE0/0/2.10               Normal   10.3.1.254     
2     Master       GE0/0/2.20               Normal   10.3.2.254 
```

##### 3. 配置抢占延迟

CE1 

```sql
int g0/0/2.10
	vrrp vrid 1 preempt-mode timer delay 60
```

CE2

```sql
int g0/0/2.20
	vrrp vrid 2 preempt-mode timer delay 60
```

#### 1.3 MSTP（5分） 

1. SW1,SW2,SW3,SW4都运行MSTP，VLAN10在instance10中，S1为 primary root，S2为 secondary root。VLAN20在instance20中，S1为  secondary root ，S2为primary root。

分析：

![image-20211005222727268](https://i.loli.net/2021/10/05/skKrGBw1QLF6CzW.png)

解法：

SW1-SW2-SW3-SW4

```sql
stp enable
stp mode mstp
stp region-configuration
	region-name HUAWEI
	instance 10 vlan 10
	instance 20 vlan 20
	active region-configuration
```

SW1 配置

```sql
stp instance 10 root primary
stp instance 20 root secondary
```

SW2 配置

```sql
stp instance 10 root secondary
stp instance 20 root primary
```

2. 了交换机互连的接口，其他接口要确保不参与MSTP计算，立刻由disable变为forwarding状态。（2） 

分析：

配置 SW3 、SW4 的边缘端口



```sql
stp bpdu-protection 
int g0/0/1
	stp edge-port enable
```

验证：

最好配置下 bpdu 保护，这样可以方便查看现象，哪个接口是 ep 端口

```sql
[SW3]disp stp instance 10 brief 
 MSTID  Port                        Role  STP State     Protection
  10    GigabitEthernet0/0/1        DESI  FORWARDING      BPDU
  10    GigabitEthernet0/0/9        ALTE  DISCARDING      NONE
  10    GigabitEthernet0/0/10       ROOT  FORWARDING      NONE
[SW3]disp stp instance 20 brief
 MSTID  Port                        Role  STP State     Protection
  20    GigabitEthernet0/0/9        ROOT  FORWARDING      NONE
  20    GigabitEthernet0/0/10       ALTE  DISCARDING      NONE
```

```sql
[SW4]disp stp instance 20 brief 
 MSTID  Port                        Role  STP State     Protection
  20    GigabitEthernet0/0/1        DESI  FORWARDING      NONE
  20    GigabitEthernet0/0/9        ALTE  DISCARDING      NONE
  20    GigabitEthernet0/0/10       ROOT  FORWARDING      NONE
```

除了 MSTID 不一样，其余都差不多

#### 1.4 WAN（2分）

1. PE1-RR1的互连的Serial接口，绑定为一个逻辑接口，成员采用HDLC。逻辑接口的IPV4,IPV6地址，按照图1，5配置。 
2. PE3-CE3的互连接口POS接口，绑定为一个逻辑接口，成员采用PPP，逻辑接口的IPV4地址，按照图1配置。

解法：

##### 1. PE1 - RR1 配置 `ip-trunk`

PE1 和 RR1 的 ser0/0/0、ser0/0/1 改为 HDLC

```sql
interface Serial0/0/0
	link-protocol hdlc  --- 输入 y 确认
interface Serial0/0/1
	link-protocol hdlc
```

PE1 配置

```sql
interface Ip-Trunk1
	ipv6 enable
	ip address 10.1.13.1 30
	ipv6 address 2000:EAD8:99EF:C03E:B2AD:9EFF:32DD:DC10/127
interface Serial0/0/0
	ip-trunk 1
interface Serial0/0/1
	ip-trunk 1
```

PR1 配置

```sql
interface Ip-Trunk1
	ipv6 enable
	ip address 10.1.13.2 30
	ipv6 address 2000:EAD8:99EF:C03E:B2AD:9EFF:32DD:DC11/127
interface Serial0/0/0
	ip-trunk 1
interface Serial0/0/1
	ip-trunk 1
```

验证：

PE1

```
ping 10.1.13.2
```

```sql
[PE1]disp int Ip-Trunk 1
-----------------------------------------------------
PortName                      Status      Weight
-----------------------------------------------------
Serial0/0/0                   UP          1
Serial0/0/1                   UP          1
-----------------------------------------------------
The Number of Ports in Trunk : 2
The Number of UP Ports in Trunk : 2
```

##### 2. PE3 和 CE3 配置 `MP-GROUP`

PE3

```sql
interface Mp-group0/0/0
	ip address 10.3.33.1 255.255.255.252 
interface Pos5/0/0
	ppp mp Mp-group 0/0/0
interface Pos6/0/0
	ppp mp Mp-group 0/0/0
```

CE3

```sql
interface Mp-group0/0/0
	ip address 10.3.33.2 255.255.255.252 
interface Pos5/0/0
	ppp mp Mp-group 0/0/0
interface Pos6/0/0
	ppp mp Mp-group 0/0/0
```

验证：

```sql
[CE3]disp ppp mp int Mp-group 0/0/0
 Mp-group is Mp-group0/0/0
 ===========Sublinks status begin======
 Pos5/0/0 physical UP,protocol UP
 Pos6/0/0 physical UP,protocol UP
 ===========Sublinks status end========
 Physical is MP, baudrate is 310000000 bps # 注意这里带宽叠加了
```

CE3

```sql
ping 10.3.33.1
```

### 2  IPV4 IGP（18分）

#### 2.1基础配置 

#### 2.2 OSPF（6分）

1. CE1和CE2之间的链路，及该两台设备的Loopback0，通告入OSPF区域。（已配） 
2. CE1的Ge0/0/2.10和Ge0/0/2.20，CE2的Ge0/0/2.10和Ge0/0/2.20：直接网段通告入OSPF区域0，但这些接口不能转发OSPF报文（2） 

CE1 配置

```sql
ospf 1 router-id 172.17.1.1 
	silent-interface GigabitEthernet0/0/2.10
	silent-interface GigabitEthernet0/0/2.20
	area 0.0.0.0 
		network 10.2.12.1 0.0.0.0 
  	network 10.3.1.1 0.0.0.0 
  	network 10.3.2.1 0.0.0.0 
  	network 172.17.1.1 0.0.0.0 
```

CE2 配置

```sql
ospf 1 router-id 172.17.1.2 
	silent-interface GigabitEthernet0/0/2.10
	silent-interface GigabitEthernet0/0/2.20
	area 0.0.0.0 
  	network 10.2.12.2 0.0.0.0 
  	network 10.3.1.2 0.0.0.0 
  	network 10.3.2.2 0.0.0.0 
  	network 172.17.1.2 0.0.0.0 
```
验证：disp ospf peer brief
			 disp ospf routing

3. RR2，P2,PE3,PE4在OSPF区域0中，cost如图2配置（已配） 

建议：首先检查 `router id`

​			 在系统视图下配置

![image-20211005235653385](https://i.loli.net/2021/10/05/fajZmrIwsK3FhAD.png)

1. PE3-PE4的OSPF链路类型为P2P。（1） 

```sql
int g0/0/0
	ospf network-type p2p
```

1. PE4上将Loopback0地址引入OSPF在AS200中，各OSPF网元到PE4 Loopback0的路由要包含内部cost。（3） 

> 1. 匹配到 172.16.1.2/32

```
ip ip-prefix PE4 index 10 permit 172.16.1.2 32 greater-equal 32 less-equal 32
```

> 2. route-policy 标记打上 tag

```sql
route-policy D permit node 10 
	if-match ip-prefix PE4 
	apply tag 172 
```

> 3. ospf 引入时候调用 route-policy 并设置为 type 1

```sql
ospf 1 
 import-route direct route-policy D type 1 
```

检查路由开销是否累加

RR2 上查看，发现 cost = 1511 已累计

```
[RR2]disp ip rou 172.16.1.2
------------------------------------------------------------------------------
Destination/Mask    Proto   Pre  Cost      Flags NextHop         Interface
     172.16.1.2/32  O_ASE   150  1511        D   10.1.91.2       GigabitEthernet
0/0/0
```



#### 2.3 ISIS

1. AS100内，loopback0和互连接口全部开启ISIS协议，其中PE1,PE2路由类型L1，区域号为49.0001，RR1，P1的路由类型L12，区域号为 49.0001；ASBR1,ASBR2路由类型为L2，区域号为49.0002，各网元system-ID唯一，cost-style为wide；cost值如图2配置。（除PE1-RR1之间的 逻辑链路，其余已配）（1）

解法： 

`disp cu con isis`

`disp cu | include isis cost`

前面在检查一下 is-level 类型，以及 wide、cost

将PE1和RR1之间IP-trunk开启IS-IS 

```
interface Ip-Trunk1 
	isis enable 1
```
检查： disp isis peer
```sql
0000.0000.0001  Ip-Trunk1          0000000002        Up   27s      L1 
```



2. RR2-P2的ISIS链路类型P2P。（1分）

```sql
int g0/0/0
	isis circuit-type p2p
```

`disp isis int g0/0/0 verbose `

3. 在RR2，P2上，ISIS和OSPF双向引入前缀为172.16.0.0/16的主机路由，被引入协议的cost要继承到后引入的协议中，P2和PE4的loopback0互访 走最优路径。配置要求有最好的扩展性。（8） 

#####  1. 双点双向

分析：

A、次优导致环路问题 

分析：在RR2上将OSPF和ISIS执行双向引入 

RR2

```sql
ospf
	import-route isis 1
isis
	import-route ospf 1
```

[P2]tracert -a 172.16.1.10 172.16.1.2 发现流量在P2和RR2之间出现环路 

```sql
[P2]tracert -a 172.16.1.10 172.16.1.2

 traceroute to  172.16.1.2(172.16.1.2), max hops: 30 ,packet length: 40,press CT
RL_C to break 

 1 10.1.91.1 20 ms  30 ms  20 ms 
 2 10.1.91.2 10 ms  10 ms  10 ms 
 ...
 29 10.1.91.1 100 ms  90 ms  100 ms 
 30 10.1.91.2 80 ms  80 ms  90 ms 
```

P2 错误优选 RR2 引入进 IS-IS 路由，导致访问 PE4 流量错误发回给 RR2，RR2 通过 OSPF 计算路由最优下一跳需要经过 P2，出现因为次优导致环路问题 

![image-20211015002122232](https://i.loli.net/2021/10/15/uHsFrAvdcnMBUDS.png)

解决方案：将O-ASE特定外部路由（tag 172）协议优先级修改为14 

解法：

RR2、P2

```sql
route-policy PRE permit node 10 
	if-match tag 172 
	apply preference 14  
ospf 1  
	preference ase route-policy PRE
```

 注意：上述配置需要在RR2、P2都进行配置 

B、配置要求有最好的扩展性 

分析：PE4 的环回口路由失效导致 P2 无法学习到对应路由，错误优选 IS-IS 然后引入回 OSPF，都可能导致路由回灌，出现新的环路问题 此处借助 `FA` 地址可以防环，收到 `LSA-5` 发现 `FA` 地址为自身则不计算路由。

PE4 

```sql
<PE4>reset ospf process
```

P2 马上

```sql
[P2]tracert -a 172.16.1.10 172.16.1.2
 traceroute to  172.16.1.2(172.16.1.2), max hops: 30 ,packet length: 40,press CT
RL_C to break 
 1 10.1.102.2 20 ms  20 ms  20 ms 
[P2]tracert -a 172.16.1.10 172.16.1.2
Oct 15 2021 00:38:40-08:00 P2 %%01OSPF/3/NBR_DOWN_REASON(l)[7]:Neighbor state le
aves full or changed to Down. (ProcessId=256, NeighborRouterId=2.1.16.172, Neigh
borAreaId=0, NeighborInterface=GigabitEthernet0/0/2,NeighborDownImmediate reason
=Neighbor Down Due to 1-Wayhello Received, NeighborDownPrimeReason=1-Wayhello Re
ceived, NeighborChangeTime=2021-10-15 00:38:40-08:00) 
[P2]tracert -a 172.16.1.10 172.16.1.2
Oct 15 2021 00:38:40-08:00 P2 %%01OSPF/4/NBR_CHANGE_E(l)[8]:Neighbor changes eve
nt: neighbor status changed. (ProcessId=256, NeighborAddress=2.102.1.10, Neighbo
rEvent=1-Way, NeighborPreviousState=Full, NeighborCurrentState=Init) 
[P2]tracert -a 172.16.1.10 172.16.1.2
 traceroute to  172.16.1.2(172.16.1.2), max hops: 30 ,packet length: 40,press CT
RL_C to break 
 1 10.1.91.1 40 ms  20 ms  30 ms 
 2 10.1.91.2 10 ms  10 ms  10 ms 
 3 10.1.91.1 20 ms  20 ms  20 ms 
 4 10.1.91.2 10 ms  10 ms  20 ms 
 5 10.1.91.1 40 ms  30 ms  30 ms 
 6 10.1.91.2 20 ms  20 ms  20 ms 
 7 10.1.91.1 40 ms  30 ms  30 ms 
 8 10.1.91.2 30 ms  20 ms  30 ms 
 9 10.1.91.1 40 ms  40 ms 
Oct 15 2021 00:38:44-08:00 P2 %%01OSPF/4/NBR_CHANGE_E(l)[9]:Neighbor changes eve
nt: neighbor status changed. (ProcessId=256, NeighborAddress=2.102.1.10, Neighbo
rEvent=2WayReceived, NeighborPreviousState=Init, NeighborCurrentState=ExStart) 
[P2]
Oct 15 2021 00:38:44-08:00 P2 %%01OSPF/4/NBR_CHANGE_E(l)[10]:Neighbor changes ev
ent: neighbor status changed. (ProcessId=256, NeighborAddress=2.102.1.10, Neighb
orEvent=NegotiationDone, NeighborPreviousState=ExStart, NeighborCurrentState=Exc
hange) 
[P2]
Oct 15 2021 00:38:44-08:00 P2 %%01OSPF/4/NBR_CHANGE_E(l)[11]:Neighbor changes ev
ent: neighbor status changed. (ProcessId=256, NeighborAddress=2.102.1.10, Neighb
orEvent=ExchangeDone, NeighborPreviousState=Exchange, NeighborCurrentState=Loadi
ng) 
[P2]
Oct 15 2021 00:38:44-08:00 P2 %%01OSPF/4/NBR_CHANGE_E(l)[12]:Neighbor changes ev
ent: neighbor status changed. (ProcessId=256, NeighborAddress=2.102.1.10, Neighb
orEvent=LoadingDone, NeighborPreviousState=Loading, NeighborCurrentState=Full) 
[P2] * 
10 10.1.102.2 30 ms  20 ms  10 ms 
```

![image-20211015004214731](https://i.loli.net/2021/10/15/m4PtezXnAESVqRZ.png)

（如果将RR2和P2之间修改为P2P则可能出现环路问题） 归根结底造成环路问题的原因是路由错误回灌，如何防止路由回灌？

> 1. 双 tag 过滤

RR2

~~~sql
ip ip-prefix 172 permit 172.16.0.0 16 g 32 l 32

route-policy O2I deny node 10
	if-match tag 10
route-policy O2I permit node 20
	if-match ip-prefix 172
	apply tag 20

route-policy I2O deny node 10
	if-match tag 30
route-policy I2O permit node 20
	if-match ip-prefix 172
	apply tag 40
~~~

P2

```sql
ip ip-prefix 172 permit 172.16.0.0 16 g 32 l 32

route-policy O2I deny node  10
	if-match tag 40
route-policy O2I permit node  20
	if-match ip-prefix 172
	apply tag 30

route-policy I2O deny node 10
	if-match tag 20
route-policy I2O permit  node 20
	if-match ip-prefix 172
	apply tag 10
```

> 2. 策略调用

RR2、PE2

~~~sql
ospf
	import isis route-policy I2O
isis
	import ospf route-policy O2I
~~~

> 3. 验证 
>
>    查看 route-policy 中是否有 deny 掉的路由

~~~sql
[RR2]disp route-policy
Route-policy : O2I
  deny : 10 (matched counts: 0)
    Match clauses : 
      if-match tag 10
  permit : 20 (matched counts: 8)
    Match clauses : 
      if-match ip-prefix 172
    Apply clauses : 
      apply tag 20 
Route-policy : I2O
  deny : 10 (matched counts: 7)  # 此处有 deny
    Match clauses : 
      if-match tag 30
  permit : 20 (matched counts: 10)
    Match clauses : 
      if-match ip-prefix 172
    Apply clauses : 
      apply tag 40 
~~~

~~~sql
[P2]disp route-policy
Route-policy : PRE
  permit : 10 (matched counts: 40)
    Match clauses : 
      if-match tag 172
    Apply clauses : 
      apply preference 14 
Route-policy : O2I
  deny : 10 (matched counts: 0)
    Match clauses : 
      if-match tag 40
  permit : 20 (matched counts: 15)
    Match clauses : 
      if-match ip-prefix 172
    Apply clauses : 
      apply tag 30 
Route-policy : I2O
  deny : 10 (matched counts: 0)
    Match clauses : 
      if-match tag 20
  permit : 20 (matched counts: 21)
    Match clauses : 
      if-match ip-prefix 172
    Apply clauses : 
      apply tag 10 
~~~

##### 2. import-route 继承 cost 开销

RR2、P2 

~~~sql
ospf 1 
 default cost inherit-metric
 import-route isis route-policy I2O type 1
isis 1
 import-route ospf 1 inherit-cost route-policy O2I 
~~~

验证

<img src="C:/Users/XuShengjin/AppData/Roaming/Typora/typora-user-images/image-20210922202548556.png" alt="image-20210922202548556" style="zoom:50%;" />

PE4 --- ASBR3 的 cost 值  `172.16.1.2/32`

1 + 1500 + 10 + 1000 = 2511

1 是 PE4 `import-route` 进 ospf 的 cost 为 1

~~~sql
<ASBR3>disp ip rou 172.16.1.2
Route Flags: R - relay, D - download to fib
------------------------------------------------------------------------------
Routing Table : Public
Summary Count : 1
Destination/Mask    Proto   Pre  Cost      Flags NextHop         Interface

    172.16.1.2/32  ISIS-L2 15   2551        D   10.1.79.2       GigabitEthernet0/0/1
~~~



##### 3. 延迟时间

4.P1的ISIS进程：产生LSP的最大的延迟时间是1S，初始延迟为50ms，递增时间为50ms，使得LSP的快速扩散特性；SPF计算最大延迟时间是1S， 初始延迟为100ms，递增时间为100ms（2）

`P1` 配置

~~~bash
isis
 timer lsp-generation 1 50 50  # 最大延迟时间	初始时间	递增时间
 flash-flood	# 快速扩散
 timer spf 1 100 100	# 最大延迟时间	初始时间	递增时间
~~~



### 3. MPLS VPN （35分）

1. CE1,CE2为VPN1的Hub-CE，PE1,PE2为Hub-PE；CE3，CE4为VPN的spoke站点；PE3，PE4为SPOKE-PE 
2. CE4为Multi-VPN-instance CE，CE4的VPN实例1，通过Ge0/0/1连接PE4。 
3. 合理设置VPN1参数，使得Spoke站点互访的流量必须经过Hub-CE设备。当CE1-PE1链路断开的情况下，PE1仍然可以学习到CE1的业务路 由。（PE3上的VPN1的RD为100:13,EXPORT RT为100：1，import RT为200：1）（2）

##### 1. AS 规划图

<img src="https://i.loli.net/2021/10/01/ih7leFNDxVsgb8R.png" alt="image-20211001220819221" style="zoom: 200%;" />



##### 2. RD、RT 值规划

![image-20210922223315260](https://i.loli.net/2021/09/22/TeKrUWdp7jtZSqI.png)

>  RD 值一样 RR 会选路，次优

![image-20210922225755402](https://i.loli.net/2021/09/22/e6UlNjSYgcFtkGb.png)

##### 3. PE 创建 VPN

PE3 配置

> 1. 配置 RD、RT 值

~~~sql
ip vpn-instance VPN1
 ipv4-family
  route-distinguisher 100:13
  vpn-target 100:1 export-extcommunity
  vpn-target 200:1 import-extcommunity
~~~

> 2. 接口绑定 VPN1

~~~sql
[PE3]int Mp-group 0/0/0
[PE3-Mp-group0/0/0]disp this
  interface Mp-group0/0/0
   ip address 10.3.33.1 255.255.255.252 
ip binding vpn-instance VPN1
ip address 10.3.33.1 255.255.255.252 
~~~

PE4 配置

> 同 PE3
~~~sql
ip vpn-instance VPN1
 ipv4-family
  route-distinguisher 100:14
  vpn-target 100:1 export-extcommunity
  vpn-target 200:1 import-extcommunity
~~~

~~~sql
interface GigabitEthernet0/0/1
 ip binding vpn-instance VPN1
 ip address 10.3.34.1 255.255.255.0 
~~~

PE1 配置

> 1. 配置两个 VPN，TOH --- 负责接收，TOS 负责发送

~~~sql
ip vpn-instance TOH
 ipv4-family
  route-distinguisher 1:1
  vpn-target 100:1 200:1 import-extcommunity
ip vpn-instance TOS
 ipv4-family
  route-distinguisher 11:11
  vpn-target 200:1 export-extcommunity
~~~

PE2 配置

~~~sql
ip vpn-instance TOH
 ipv4-family
  route-distinguisher 2:2
  vpn-target 100:1 200:1 import-extcommunity
ip vpn-instance TOS
 ipv4-family
  route-distinguisher 22:22
  vpn-target 200:1 export-extcommunity
~~~



4. CE1通过G0/0/1.1和G0/0/1.2建立直接EBGP邻居，接入PE1。CE1通过G0/0/1.2,向PE1通告BGP update中，某些路由信息的ASpath中有200。在CE1上，将OSPF路由导入BGP。（2）
5. CE2通过G0/0/1.1和G0/0/1.2建立直接EBGP邻居，接入PE2。CE2通过G0/0/1.2,向PE2通告BGP update中，某些路由信息的AS-path中有 200。在CE2上，将OSPF导入BGP。（2） 

分析：

​	也就是约定了 CE1 哪个子接口绑定哪个 `VPN` 实例

![image-20211001212513690](https://i.loli.net/2021/10/01/rLYqVn9GgjmDk7a.png)

##### 4. CE --- PE 配置 VPNv4 邻居

解法：

IP

| 设备 | 网段         | 编号     |
| ---- | ------------ | -------- |
| CE1  | 10.2.11.0/30 | .2    .6 |
| PE1  | 10.2.11.0/30 | .1    .5 |
| CE2  | 10.2.22.0/30 | .2    .6 |
| PE2  | 10.2.22.0/30 | .1    .5 |

CE1 配置

~~~sql
interface GigabitEthernet0/0/1.1
 ip address 10.2.11.2 255.255.255.252 
interface GigabitEthernet0/0/1.2
 ip address 10.2.11.6 255.255.255.252 
bgp 65000
 peer 10.2.11.1 as-number 100 
 peer 10.2.11.5 as-number 100 
 import ospf 1
~~~

CE2 配置

```sql
interface GigabitEthernet0/0/1.1
 ip address 10.2.22.2 255.255.255.252 
interface GigabitEthernet0/0/1.2
 ip address 10.2.22.6 255.255.255.252 
bgp 65000
 peer 10.2.22.1 as-number 100 
 peer 10.2.22.5 as-number 100 
 import ospf 1
```

PE1 配置

> 两个子接口 分别维护两个 虚拟路由
>
> 基于两个 vpn 去建立对等体

```sql
interface GigabitEthernet0/0/1.1
 ip binding vpn-instance TOH
 ip address 10.2.11.1 255.255.255.252
interface GigabitEthernet0/0/1.2
 ip binding vpn-instance TOS
 ip address 10.2.11.5 255.255.255.252
 
bgp 100
ipv4-family vpn-instance TOH
  peer 10.2.11.2 as-number 65000

ipv4-family vpn-instance TOS
  peer 10.2.11.6 as-number 65000
```

PE2 配置

```sql
interface GigabitEthernet0/0/1.1
 ip binding vpn-instance TOH
 ip address 10.2.22.1 255.255.255.252
interface GigabitEthernet0/0/1.2
 ip binding vpn-instance TOS
 ip address 10.2.22.5 255.255.255.252
 
bgp 100
ipv4-family vpn-instance TOH
  peer 10.2.22.2 as-number 65000

ipv4-family vpn-instance TOS
  peer 10.2.22.6 as-number 65000
```

验证

PE1、PE2：

```sql
[PE1]disp bgp vpnv4 all peer

 BGP local router ID : 10.1.13.1
  Peer of IPv4-family for vpn instance :
 VPN-Instance TOH, Router ID 10.1.13.1:
  10.2.11.2       4       65000        5        5     0 00:03:54 Established    0
 VPN-Instance TOS, Router ID 10.1.13.1:
  10.2.11.6       4       65000        3        3     0 00:01:20 Established    0

```

VPNv4 路由

`disp bgp vpnv4 all routing-table`

```
 *>   10.2.12.0/30       10.2.11.2       0                     0      65000?
 *>   10.3.1.0/24        10.2.11.2       0                     0      65000?
 *>   10.3.2.0/24        10.2.11.2       0                     0      65000?
 *>   172.17.1.1/32      10.2.11.2       0                     0      65000?
 *>   172.17.1.2/32      10.2.11.2       1  
```



6. CE3通过OSPF区域1接入PE3，通过PE3-CE3的逻辑接口互通，通告CE3的各环回口；CE4通过OSPF区域0接入PE4，通过PE4-CE4的Ge0/0/1 接口互通，通告 CE4的各环回口；（2） 

解法：

​	注意在之前配置 `vpn` 时候接口已经绑定的 `vpn instance` 了。 

PE3 配置

```sql
ospf 10 vpn-instance VPN1
	area 1
		network 10.3.33.1 0.0.0.0  
		# Mp-group0/0/0
```

CE3 配置

```sql
ospf 10
	area 1
		network 10.3.33.2 0.0.0.0
		network 172.17.1.3 0.0.0.0
		network 10.3.3.3 0.0.0.0
```

PE4 配置

```sql
ospf 10 vpn-instance VPN1
	area 0
		network 10.3.34.1 0.0.0.0  
```

正常 `CE4` 配置

​	配置地址

```sql
interface GigabitEthernet0/0/1
	ip address 10.3.34.2 255.255.255.0 
interface LoopBack0
	ip address 172.17.1.4 255.255.255.255 
interface LoopBack1
	ip address 10.3.4.4 255.255.255.255 
```

```sql
ospf 10
	area 0
		network 10.3.34.2 0.0.0.0
		network 172.17.1.4 0.0.0.0
		network 10.3.4.4 0.0.0.0
```

若是 `MCE` 配置

![image-20211002132013706](https://i.loli.net/2021/10/02/Pjc7vkwoalBesA8.png)

> 需要绑定实例

```sql
ip vpn-instance VPN1
 ipv4-family
  route-distinguisher 100:14
interface GigabitEthernet0/0/1
	ip binding vpn-instance VPN1
	ip address 10.3.34.2 255.255.255.252
interface LoopBack0
	ip binding vpn-instance VPN1
	ip address 172.17.1.4 255.255.255.255 
interface LoopBack1
	ip binding vpn-instance VPN1
	ip address 10.3.4.4 255.255.255.255 
```

```sql
ospf 10 vpn-instance VPN1
	area 0
		network 10.3.34.2 0.0.0.0
		network 172.17.1.4 0.0.0.0
		network 10.3.4.4 0.0.0.0
```

双 `PE` 防环，设置 `DN` 位 或者 `route-tag` 来防环

![image-20211002132034967](https://i.loli.net/2021/10/02/1iCPkj8bJm4MOaw.png)

`MCE` 需关闭 `DN` 位

```sql
ospf 10 vpn-instance VPN1
	vpn-instance-capability simple
```

问题：

​	为什么有了 `DN` 位，还要设定 `route-tag` ？它主要用在什么场景？

验证：检查 vpn-instance VPN1 路由

​	PE4

```sql
[PE4]disp ip rou vpn-instance VPN1
 10.3.4.4/32  OSPF    10   1           D   10.3.34.2       GigabitEthernet0/0/1
 172.17.1.4/32  OSPF    10   1           D   10.3.34.2       GigabitEthernet0/0/1
 10.3.34.1/32  Direct  0    0           D   127.0.0.1       GigabitEthernet0/0/1
```



7. 在AS100，AS200内建立IBGP IPV4邻居关系，RR1是PE1,PE2,P1,ASBR1,ASBR2的反射器，RR2是PE3，PE4，P2，ASBR4的反射 器。ASBR1-ASBR3，ASBR2-ASBR4建立EBGP IPV4邻居关系。（已预配） 

8. 在ASBR上，将ISIS的loopback0路由引入BGP（2）

##### 5. ASBR 上发布路由

解法：

​	ASBR1 、ASBR2

```sql
ip ip-prefix 172 index 10 permit 172.16.0.0 16 greater-equal 32 less-equal 32
route-policy I2B permit node 10
	if-match ip-prefix 172
bgp 100
	import-route isis 1 route-policy I2B
```

​	ASBR3 、 ASBR4

```sql
ip ip-prefix 172 index 10 permit 172.16.0.0 16 greater-equal 32 less-equal 32
route-policy I2B permit node 10
	if-match ip-prefix 172
bgp 200
	import-route isis 1 route-policy I2B
```

检查

​	ASBR1 and ASBR4 上 查看是否有对端 `bgp` 路由

​	[ASBR1]disp bgp routing-table

![image-20211002135603068](https://i.loli.net/2021/10/02/tieyqZ4cpoz7krA.png)

[ASBR4]disp bgp routing-table

![image-20211002135647446](https://i.loli.net/2021/10/02/4AmPVhlbeuxyJ1q.png)

建议：配置 `ASBR` 指向  `RR` 配置 `next-hop-local`

因为 LAB2 必配，不用纠结了

为什么 `lab1` 不用配、`lab2` 必配，分析

```
bgp 100
	peer 172.16.1.3 next-hop-local
```

```
bgp 200
	peer 172.16.1.9 next-hop-local
```



9.  AS100，AS200内各网元配置MPLS LSR-ID，全局使能MPLS,MPLS LDP（已配）AS100,AS200内各有直连链路建立LDP邻居（除 PE1-RR1之间，其余已配）（1）

PE1、RR1 配置

```sql
interface Ip-Trunk1q
 mpls
 mpls ldp
```

验证：

RR1 看到有 3 条会话就 ok 了

```sql
[RR1]disp mpls ldp session all
 ------------------------------------------------------------------------------
 PeerID             Status      LAM  SsnRole  SsnAge      KASent/Rcv
 ------------------------------------------------------------------------------
 172.16.1.1:0       Operational DU   Active   0000:00:00  1/1
 172.16.1.4:0       Operational DU   Passive  0000:01:29  357/357
 172.16.1.5:0       Operational DU   Passive  0000:01:29  357/357
 ------------------------------------------------------------------------------
 TOTAL: 3 session(s) Found.
```

##### 6. OPTION C方案一

10. 如图4，各站点，通过MPLS BGP VPN跨域OPTION C方案一，能够相互学习路由，MPLS域不能出现次优路径。（15）

分析：

![image-20211002172530443](https://i.loli.net/2021/10/02/1MQg6n3BEq8LDCR.png)

> 1. PE3 和 PE4 将 OSPF 和 BGP 双向引入

```sql
bgp 200
	ipv4-family vpn-instance VPN1	
		import-route ospf 10
ospf 10
	import-route bgp
```

> 2. AS 内部 PE 和 RR 之间 VPNv4 的对等体

**PE1 、PE2**

```sql
bgp 100
	ipv4-family vpnv4
		peer 172.16.1.3 enable
```

RR1

```sql
bgp 100
	ipv4-family vpnv4pp
  	policy vpn-target
  	peer 172.16.1.1 enable
  	peer 172.16.1.1 reflect-client
  	peer 172.16.1.20 enable
  	peer 172.16.1.20 reflect-client
```

RR1 关闭 `RT` 值检测

```
bgp 100
	ipv4-family vpnv4
  undo policy vpn-target
```

**PE3 、 PE4**

```sql
bgp 200
	ipv4-family vpnv4
		peer 172.16.1.9 enable
```

RR2

```sql
bgp 200
	ipv4-family vpnv4
  	policy vpn-target
  	peer 172.16.1.11 enable
  	peer 172.16.1.11 reflect-client
  	peer 172.16.1.2 enable
  	peer 172.16.1.2 reflect-client
```

关闭 `RT` 值检测, RR 你只是帮忙转发一下，真正要检测的是 PE 去做的

```
bgp 200
	ipv4-family vpnv4
		undo policy vpn-target
```

检查：

​	RR1、RR2：disp bgp vpnv4 all peer 、disp bgp vpnv4 all routing-table

> 问题：为什么要让 PE 把路由传给 RR 呢？
>

​	以前都是 PE 和 CE 之间互传路由，中间的 RR、ASBR 都不需要传递 VPNv4 路由，那是简单的跨域需求

​	但是现在的需求是，需要多个站点间互访，就不能这么干了。

![image-20211002173124517](https://i.loli.net/2021/10/02/bCUfD5L4wxIugiA.png)

现在的做法呢，只需要 PE 把路由传递给 RR，然后两个 RR 之间建立一个 跨域的`多跳 MP_EBGP`

![image-20211002173604689](https://i.loli.net/2021/10/02/tcGoHM2pSRFU43J.png)



> 3. RR 之间建立一个 跨域的`多跳 MP_EBGP`，传递 VPNv4 路由

RR1 配置

```sql
bgp 100
	peer 172.16.1.9 as-number 200
	peer 172.16.1.9 ebgp-max-hop 255
	peer 172.16.1.9 connect-interface LoopBack0
 
	ipv4-family vpnv4
 		peer 172.16.1.9 enable
```

**建议：关闭 RR 之间的 IPv4 单播路由传递能力，只传递 VPNv4 路由**

![image-20211002212747947](https://i.loli.net/2021/10/02/4rikBt8yX51FKqO.png)

原因：查看 ASBR 表项的时候会有选路问题、查表失败

```sql
bgp 100 
	ipv4-family unicast
 		undo peer 172.16.1.9 enable
```

RR2配置

```sql
bgp 200
	peer 172.16.1.3 as-number 100
	peer 172.16.1.3 ebgp-max-hop 255
	peer 172.16.1.3 connect-interface LoopBack0
 
	ipv4-family vpnv4
 		peer 172.16.1.3 enable
```

建议：关闭 RR 之间的单播路由传递

```sql
bgp 200 
	ipv4-family unicast
 		undo peer 172.16.1.3 enable
```

检查：

RR1 RR2：disp bgp vpnv4 all peer、disp bgp vpnv4 all routing

我丢，为啥这里的邻居建立花了好多分钟 --- 其实没有

RR1 为例查看

![image-20211002180801994](https://i.loli.net/2021/10/02/mFLdNO382ua9xG1.png)

> 4. 建立 PE 之间完整的 LSP，此处采用 方案一

分析：

> ​	A. ASBR 之间通过 BGP-Label Unicast，为自身单播分配并通过标签

ASBR1 配置

```sql
route-policy ASBR permit node 10
	apply mpls-label 
bgp 100
	peer 10.1.57.2 route-policy ASBR export
	peer 10.1.57.2 label-route-capability
```

ASBR3 配置

```sql
route-policy ASBR permit node 10
	apply mpls-label 
bgp 200
	peer 10.1.57.1 route-policy ASBR export
	peer 10.1.57.1 label-route-capability
```



ASBR2 配置

```sql
route-policy ASBR permit node 10
	apply mpls-label 
bgp 100
	peer 10.1.68.2 route-policy ASBR export
	peer 10.1.68.2 label-route-capability
```

ASBR4 配置

```sql
route-policy ASBR permit node 10
	apply mpls-label 
bgp 200
	peer 10.1.68.1 route-policy ASBR export
	peer 10.1.68.1 label-route-capability
```

检查：

ASBR1

```sql
[ASBR1]disp bgp routing-table label 
 Total Number of Routes: 14									分配/学习
        Network           NextHop           In/Out Label
 *>     172.16.1.1        10.1.35.1         1030/NULL
 *>     172.16.1.2        10.1.57.2         NULL/1034
 *>     172.16.1.3        10.1.35.1         1029/NULL
 *>     172.16.1.4        10.1.35.1         1032/NULL
 * i                      172.16.1.6        
 *>     172.16.1.5        127.0.0.1         1034/NULL
 * i                      172.16.1.6        
 *>     172.16.1.6        10.1.56.2         1033/NULL
 *>     172.16.1.7        10.1.57.2         NULL/1035
 *>     172.16.1.8        10.1.57.2         NULL/1030
 *>     172.16.1.9        10.1.57.2         NULL/1031
 *>     172.16.1.10       10.1.57.2         NULL/1032
 *>     172.16.1.11       10.1.57.2         NULL/1033
 *>     172.16.1.20       10.1.35.1         1031/NULL
```

 ASBR3 

```sql
[ASBR3]disp bgp routing-table label 
Total Number of Routes: 15									分配/学习
				Network           NextHop           In/Out Label

 *>     172.16.1.1        10.1.57.1         NULL/1030
 *>     172.16.1.2        10.1.79.2         1034/NULL
 * i                      172.16.1.8        
 *>     172.16.1.3        10.1.57.1         NULL/1029
 *>     172.16.1.4        10.1.57.1         NULL/1032
 *>     172.16.1.5        10.1.57.1         NULL/1034
 *>     172.16.1.6        10.1.57.1         NULL/1033
 *>     172.16.1.7        127.0.0.1         1035/NULL
 * i                      172.16.1.8        
 *>     172.16.1.8        10.1.78.2         1030/NULL
 *>     172.16.1.9        10.1.79.2         1031/NULL
 *>     172.16.1.10       10.1.79.2         1032/NULL
 * i                      172.16.1.8        
 *>     172.16.1.11       10.1.79.2         1033/NULL
 *>     172.16.1.20       10.1.57.1         NULL/1031
```

> B.  ASBR 重分配 label，继续通告给本端 RR（由 RR 反射给 PE 设备）

ASBR1 配置

```sql
route-policy RR permit node 10
	if-match mpls-label
	apply mpls-label
bgp 100
	peer 172.16.1.3 route-policy RR export
	peer 172.16.1.3 label-route-capability
	# 隐藏 next-hop-local
```

ASBR3 配置

```sql
route-policy RR permit node 10
	if-match mpls-label
	apply mpls-label
bgp 200
	peer 172.16.1.9 route-policy RR export
	peer 172.16.1.9 label-route-capability
```



ASBR2 配置

```sql
route-policy RR permit node 10
	if-match mpls-label
	apply mpls-label
bgp 100
	peer 172.16.1.3 route-policy RR export
	peer 172.16.1.3 label-route-capability
	# 隐藏 next-hop-local
```

ASBR4 配置

```sql
route-policy RR permit node 10
	if-match mpls-label
	apply mpls-label
bgp 200
	peer 172.16.1.9 route-policy RR export
	peer 172.16.1.9 label-route-capability
```



> C. RR 、ASBR 和 PE 之间都需要开启 标签通告能力

![image-20211002211536122](https://i.loli.net/2021/10/02/frMyJACm35WPtEU.png)

RR1 配置

```sql
bgp 100
	peer 172.16.1.5 label-route-capability
	peer 172.16.1.6 label-route-capability
	peer 172.16.1.1 label-route-capability
	peer 172.16.1.20 label-route-capability
```

PE1 、PE2 配置

```sql
bgp 100
	peer 172.16.1.3 label-route-capability
```

RR2 配置

```sql
bgp 200
	peer 172.16.1.7 label-route-capability
	peer 172.16.1.8 label-route-capability
  peer 172.16.1.11 label-route-capability
	peer 172.16.1.2 label-route-capability
```

PE3 、PE4 配置

```sql
bgp 200
	peer 172.16.1.9 label-route-capability
```



但此时 RR 上查看 `bgp-label` 还是没有，怎么回事呢？发现 ASBR 之间的接口**没有开启 `mpls` 功能** --- <font color='red'>建议放在 IP-TRUNK 1 那里直接配了</font>

![image-20211002211255441](https://i.loli.net/2021/10/02/BzlDT7oijAre6dc.png)

ASBR1-4

```
int g0/0/2
	mpls
```

此时 ASBR、RR 上再查看

以 ASBR1、RR1 为例

已经可以看到 为对端路由 重新分配标签了

```sql
[ASBR1]disp bgp routing-table label
				Network           NextHop           In/Out Label
 *>     172.16.1.1        10.1.35.1         1030/NULL
 * i                      172.16.1.6        
 *>     172.16.1.2        10.1.57.2         1036/1034
 *>     172.16.1.3        10.1.35.1         1029/NULL
 * i                      172.16.1.6        
 *>     172.16.1.4        10.1.35.1         1032/NULL
 * i                      172.16.1.6        
 *>     172.16.1.5        127.0.0.1         1034/NULL
 * i                      172.16.1.6        
 *>     172.16.1.6        10.1.56.2         1033/NULL
 *>     172.16.1.7        10.1.57.2         1035/1035
 *>     172.16.1.8        10.1.57.2         1040/1030
 *>     172.16.1.9        10.1.57.2         1039/1031
 *>     172.16.1.10       10.1.57.2         1038/1032
 *>     172.16.1.11       10.1.57.2         1037/1033
 *>     172.16.1.20       10.1.35.1         1031/NULL
 * i                      172.16.1.6        
```
RR1 也收到了对端的路由了

```sql
[RR1-bgp]disp bgp routing-table label
        Network           NextHop           In/Out Label

 *>i    172.16.1.2        172.16.1.5        NULL/1036
 *>i    172.16.1.7        172.16.1.5        NULL/1035
 *>i    172.16.1.8        172.16.1.5        NULL/1040
 *>i    172.16.1.9        172.16.1.5        NULL/1039
 *>i    172.16.1.10       172.16.1.5        NULL/1038
 *>i    172.16.1.11       172.16.1.5        NULL/1037
```

> D. AS100 中因为存在 Level-1 设备没有办法学习到 ASBR1、ASBR2 的明细路由，导致 BGP 标签路由无效，需要配置路由泄露 
>

RR1 、P1配置 

```sql
ip ip-prefix 172 index 10 permit 172.16.0.0 16 greater-equal 32 less-equal 32 
isis 1 
	import-route isis level-2 into level-1 filter-policy ip-prefix 172
```

验证

此时跨域以及完成了，CE1 --- CE3， CE2 --- CE4，CE1 --- CE4， CE2 --- CE3

```sql
[CE3]ping -a 172.17.1.3 172.17.1.1
  PING 172.17.1.1: 56  data bytes, press CTRL_C to break
    Reply from 172.17.1.1: bytes=56 Sequence=1 ttl=250 time=150 ms
    Reply from 172.17.1.1: bytes=56 Sequence=2 ttl=250 time=120 ms
    Reply from 172.17.1.1: bytes=56 Sequence=3 ttl=250 time=130 ms
    Reply from 172.17.1.1: bytes=56 Sequence=4 ttl=250 time=130 ms
    Reply from 172.17.1.1: bytes=56 Sequence=5 ttl=250 time=100 ms
```

但是我要访问的是 分部 CE3 --- CE4

```sql
[CE3]ping -a 172.17.1.3 172.17.1.4
  PING 172.17.1.4: 56  data bytes, press CTRL_C to break
    Request time out
    Request time out
    Request time out
    Request time out
    Request time out
```



> E. 实现 Hub-Spoke 路由正确学习，配置允许 AS 号重复
>

![image-20211004171114992](https://i.loli.net/2021/10/04/PS2YGEc7Hoz6eZv.png)

PE1 配置

```sql
bgp 100
	ipv4-family vpn-instance TOS
 	  peer 10.2.11.6 allow-as-loop
```

PE2  配置

```sql
bgp 100
	ipv4-family vpn-instance TOS
  	peer 10.2.22.6 allow-as-loop
```

RR2 配置

```sql
bgp 200
	ipv4-family vpnv4
  	peer 172.16.1.3 allow-as-loop
```

CE3 --- CE4 

```sql
ping -a 172.17.1.3 172.17.1.4
  PING 172.17.1.4: 56  data bytes, press CTRL_C to break
    Reply from 172.17.1.4: bytes=56 Sequence=1 ttl=250 time=200 ms
    Reply from 172.17.1.4: bytes=56 Sequence=2 ttl=250 time=210 ms
    Reply from 172.17.1.4: bytes=56 Sequence=3 ttl=250 time=190 ms
    Reply from 172.17.1.4: bytes=56 Sequence=4 ttl=250 time=210 ms
    Reply from 172.17.1.4: bytes=56 Sequence=5 ttl=250 time=230 ms
```

发现通了，tracert 看一下，发现有断层

```sql
tracert -a 172.17.1.3 172.17.1.4
	1 10.3.33.1 30 ms  1 ms  10 ms 
	2  *  *  * 
	3 10.1.79.1 100 ms  100 ms  110 ms 
	4 10.1.57.1 130 ms  100 ms  110 ms 
	5  *  *  * 
	6 10.2.11.5 100 ms  110 ms  110 ms 
	7 10.2.11.6 110 ms  90 ms  100 ms 
	8 10.2.11.1 110 ms  100 ms  110 ms 
	9 10.3.34.2 190 ms  190 ms  190 ms
```

CE4 --- CE2

```sql
tracert -a 172.17.1.4 172.17.1.2
 1 10.3.34.1 30 ms  20 ms  10 ms 
 2 10.1.102.1 140 ms  100 ms  140 ms 
 3  *  *  * 
 4 10.1.79.1 110 ms  160 ms  110 ms 
 5 10.1.57.1 120 ms  130 ms  130 ms 
 6  *  *  * 
 7 10.1.13.1 120 ms  140 ms  140 ms 
 8 10.2.22.5 110 ms  130 ms  130 ms 
 9 10.2.22.6 140 ms  110 ms  130 ms
```

发现次优了

![image-20211006125349933](https://i.loli.net/2021/10/06/vsH7rygSwBDlELt.png)

发现中间断了，这是没有建立完整的 `LSP` 通道的原因

> F. MPLS 域内不能出现次优问题！

MPLS-VPN 传递路由 EBGP 传递给 IBGP 时候会自动更改下一跳，所以额外需要配置下一跳不变

![image-20211004172149046](https://i.loli.net/2021/10/04/pzb1uAfWgr8J2BF.png)

RR1 配置

```sql
bgp 100
	ipv4-family vpnv4
		peer 172.16.1.9 next-hop-invariable
		peer 172.16.1.1 next-hop-invariable
		peer 172.16.1.20 next-hop-invariable
```

RR2 配置

```sql
bgp 200
	ipv4-family vpnv4
		peer 172.16.1.3 next-hop-invariable
		peer 172.16.1.11 next-hop-invariable
		peer 172.16.1.2 next-hop-invariable
```

在验证前建议在 `PE` 上配置：

用来使能 MPLS VPN 报文的 IP TTL复制功能

```sql
mpls
	ttl propagate vpn
```

验证：完美的 `15` 跳

```sql
[PE3]tracert -a 172.17.1.3 172.17.1.4
traceroute to  172.17.1.4(172.17.1.4), max hops: 30 ,packet length: 40,press CT
RL_C to break 
1 10.3.33.1 20 ms  20 ms  20 ms 
2 10.1.119.1 110 ms  100 ms  110 ms 
3 10.1.79.1 100 ms  110 ms  110 ms 
4 10.1.57.1 120 ms  130 ms  100 ms 
5 10.1.35.1 70 ms  100 ms  110 ms 
6 10.2.11.5 110 ms  100 ms  110 ms 
7 10.2.11.6 110 ms  100 ms  110 ms  # CE1
8 10.2.11.1 90 ms  120 ms  80 ms    # PE1
9 10.1.12.2 210 ms  210 ms  210 ms  # PE2
10 10.1.24.2 230 ms  210 ms  210 ms # P1
12 10.1.68.2 200 ms  250 ms  200 ms 
13 10.1.81.2 190 ms  210 ms  230 ms 
14 10.3.34.1 190 ms  210 ms  210 ms 
15 10.3.34.2 170 ms  240 ms  190 ms
```

```sql
[CE4]tracert -vpn-instance VPN1 -a 172.17.1.4 172.17.1.3
 traceroute to VPN1 17
2.17.1.3(172.17.1.3), max hops: 30 ,packet length: 40,press CTRL_C to break 
 1 10.3.34.1 20 ms  20 ms  20 ms 
 2 10.1.102.1 120 ms  130 ms  120 ms 
 3 10.1.81.1 160 ms  100 ms  130 ms 
 4 10.1.68.1 120 ms  130 ms  150 ms 
 5 10.1.46.1 130 ms  130 ms  110 ms 
 6 10.2.22.5 120 ms  110 ms  120 ms 
 7 10.2.22.6 120 ms  130 ms  130 ms # CE2 
 8 10.2.22.1 120 ms  130 ms  120 ms # PE2
 9 10.1.12.1 230 ms  230 ms  230 ms # PE1
10 10.1.13.2 200 ms  240 ms  250 ms # RR1
11 10.1.35.2 200 ms  250 ms  220 ms 
12 10.1.57.2 260 ms  220 ms  270 ms 
13 10.1.79.2 220 ms  200 ms  220 ms 
14 10.3.33.1 210 ms  200 ms  260 ms 
15 10.3.33.2 230 ms  260 ms  210 ms 
```



![](https://i.loli.net/2021/10/05/zs7j61virVwD9IH.png)


9. CE1-PE1之间链路开，CE1设备仍可以学习到spoke业务网段。配置保障有最好的扩展性。（6） 
10. 在拓扑正常情况下，要求CE1，CE2访问spoke网段时，不从本AS内绕行。（1） 

分析

![image-20211004221915170](https://i.loli.net/2021/10/04/4EAaknGd2bO5CXH.png)

`CE` 上 `bgp` 和 `ospf` 双向引入，不会导致环路问题，但是会导致路由回灌-路由震荡问题，次优问题

不会导致环路问题：从 RR2 传来了 `172.17.1.3/32` 路由，通过 RR1 反射给 PE2 - CE2 - CE1 - PE1 -RR1 - PE2 - CE2，到 CE2 的时候 AS 号为 `100 65000`, 因为有 `65000`， 所以 CE2 不收，所以不会导致环路问题

路由震荡问题：

① 号路由为正常 从 RR2 传来了 `172.17.1.3/32` 路由，通过 RR1 反射给 PE2 nh = PE3（RR1 传来）

② 号路由为从 RR2 传来了 `172.17.1.3/32` 路由，通过 RR1 反射给 PE2 - CE2 - CE1 - PE1 - RR1 - PE2

​		nh = PE1 

PE2 视角

| 序号 | 路由          | 下一跳          | 路径                             | as-path    | 优选        |
| ---- | ------------- | --------------- | -------------------------------- | ---------- | ----------- |
| 1    | 172.17.1.3/32 | 172.16.1.3(PE3) | RR1 - PE2                        | 100        |             |
| 2    | 172.17.1.3/32 | 172.16.1.1(PE1) | PE2 - CE2 - CE1 - PE1 -RR1 - PE2 | 65000，100 | Y（离得近） |
|      |               |                 |                                  |            |             |

此时 PE2 会选路，as 号都是一样的，比较 nh 哪个离自己更近（igp 开销小），此时 会认为是 PE1 的 ②号路由更近

然后 ②号路由传给 CE2，CE2 不收，此时 ②号路由不传了，①号路由占上风了，然后就这么来来回回翻动

解决方案：给路由打 `tag`

注意：需要在 `PE1 - RR1` 的 `ip-trunk 1` 链路加上 isis cost 配置，否则后续 CE4 - CE3 选路不正确

```sql
int Ip-Trunk1
	isis cost 1500
```



CE1 配置

```sql
ospf
	import-route bgp tag 172
	
route-policy O2B deny node 10 
	if-match tag 172
route-policy O2B permit node 20  

bgp 65000 
  import-route ospf 1 route-policy O2B
```

CE2 配置

```sql
route-policy O2B deny node 10 
	if-match tag 172
route-policy O2B permit node 20  

bgp 65000 
  import-route ospf 1 route-policy O2B
  
ospf
	import-route bgp tag 172
```

> 修改 EBGP 优先级

CE1、CE2

```sql
bgp 65000
	preference 120 255 255
						EBGP IBGP LOCAL
```

验证：

```sql
[CE1-bgp]disp ip routing-table 172.17.1.3
------------------------------------------------------------------------------
Destination/Mask    Proto   Pre  Cost      Flags NextHop         Interface
     172.17.1.3/32  EBGP    120  0           D   10.2.11.1       GigabitEthernet
0/0/1.1
```

验证：CE1、CE2 下查看到有 deny

```sql
[CE1]disp route-policy
Route-policy : O2B
  deny : 10 (matched counts: 5)
    Match clauses : 
      if-match tag 172
  permit : 20 (matched counts: 5)
```

11. 在PE3，PE4上修改BGP local-preference属性为120，实现CE3,CE4访问非直接的10.3.x.0/24网段时，

    若X为奇数，PE3，PE4优选的下一跳为 PE1；

    若X为偶数，PE3，PE4优选的下一跳为PE2， 不用考虑来回路径是否一致。（3分）

PE3、PE4  配置

```sql
10.3.1.x PE1
10.3.2.x PE2
```

配置思路

通过修改 `local-preference`来选路

如果是 奇数路由，且下一跳为 PE1(172.16.1.1) 则修改  `local-preference` 为 120

如果是 偶数路由，且下一跳为 PE2(172.16.1.20) 则修改  `local-preference` 为 120

> 1. 先用 acl 匹配出 奇偶路由

```sql
acl 2001 
	rule 5 permit source 10.3.1.0 0.0.254.0 
acl 2002 
	rule 5 permit source 10.3.0.0 0.0.254.0
```

> 2. 再用 ip-prefix 匹配出 PE1、PE2

```sql
ip ip-prefix PE1 index 10 permit 172.16.1.1 32
ip ip-prefix PE2 index 10 permit 172.16.1.20 32
```

> 3. 通过 route-policy 修改 local-preference

```sql
route-policy LP permit node 10 
	if-match acl 2001 
	if-match ip next-hop ip-prefix PE1 
	apply local-preference 120 
route-policy LP permit node 20 
	if-match acl 2002 
	if-match ip next-hop ip-prefix PE2 
	apply local-preference 120 
route-policy LP permit node 30 
```

> 4. 对 RR2 入向调用

```sql
bgp 200
	ipv4-family vpnv4
 		peer 172.16.1.9 route-policy LP import
```




```sql
ip ip-prefix PE1 index 10 permit 172.16.1.1 32
ip ip-prefix PE2 index 10 permit 172.16.1.20 32
acl 2001 
	rule 5 permit source 10.3.1.0 0.0.254.0 
acl 2002 
	rule 5 permit source 10.3.0.0 0.0.254.0
route-policy LP permit node 10 
	if-match acl 2001 
	if-match ip next-hop ip-prefix PE1 
	apply local-preference 120 
route-policy LP permit node 20 
	if-match acl 2002 
	if-match ip next-hop ip-prefix PE2 
	apply local-preference 120 
route-policy LP permit node 30 
bgp 200
	ipv4-family vpnv4
 		peer 172.16.1.9 route-policy LP import
```

验证：

PE3 、PE4 上查看

```sql
disp bgp vpnv4 all routing-table 
 *>i  10.3.1.0/24        172.16.1.1                 120        0      100 65000?
 * i                     172.16.1.20                100        0      100 65000?
 *>i  10.3.2.0/24        172.16.1.20                120        0      100 65000?
 * i                     172.16.1.1                 100        0      100 65000?
```



### 4.Future（17分）

#### 4.1 HA（8分）

1. CE1配置静态的默认路由访问ISP，下一跳IP为100.0.1.2，该默认路由要与CE1-ISP链路的BFD状态绑定（CE1的对端设备不支持BFD），感受 故障时间要小于150ms（2） 

> 1. 配置 CE1 单臂回声

```sql
bfd
q
bfd toisp bind peer-ip 100.0.1.2 interface GigabitEthernet2/0/0 one-arm-echo
	discriminator local 1 # 配置本地标识符
	commit
```

验证：

```sql
[CE1]disp bfd session all
--------------------------------------------------------------------------------
Local Remote     PeerIpAddr      State     Type        InterfaceName            
--------------------------------------------------------------------------------

1     -          100.0.1.2       Up        S_IP_IF     GigabitEthernet2/0/0     
--------------------------------------------------------------------------------
     Total UP/DOWN Session Number : 1/0
```

课后问题：怎么构造 `upd` 报文？抓包观察，源目 ip地址，端口号

> 2. 绑定 BFD 会话

CE1 

```sql
ip route-static 0.0.0.0 0 100.0.1.2 track bfd-session toisp
```

验证：

```sql
[CE1]disp ip rou pro static 
Route Flags: R - relay, D - download to fib
------------------------------------------------------------------------------
Static routing table status : <Active>
         Destinations : 1        Routes : 1

Destination/Mask    Proto   Pre  Cost      Flags NextHop         Interface
        0.0.0.0/0   Static  60   0          RD   100.0.1.2       GigabitEthernet2/0/0
```

> ​	3.  配置感受故障时间 --- 忘记写

CE1

```sql
bfp toisp
	min-echo-rx-interval 40			# 因为故障次数为 3 次，3 * 40 = 120 ms
	commit
```



2. CE2，CE3,CE4能够通过默认路由访问ISP（4） 

**将默认路由通告给 CE2、CE3、CE4**

给 CE2 下发默认路由

CE1 配置

```
ospf 
	default-route-advertise
```



CE3、CE4 借助  `BGP TOS `邻居 通告给 PE1，PE1 借助 `MP-BGP`  传给 `PE3、PE4` 默认路由

CE1 配置

```sql
bgp 65000
	peer 10.2.11.5 default-route-advertise
```

PE3、PE4

```sql
<PE4>disp ip rou vpn-instance VPN1 0.0.0.0
Route Flags: R - relay, D - download to fib
------------------------------------------------------------------------------
Routing Table : VPN1
Summary Count : 1
Destination/Mask    Proto   Pre  Cost      Flags NextHop         Interface
        0.0.0.0/0   IBGP    255  0          RD   172.16.1.1      GigabitEthernet
0/0/2
```

但是此时 CE3、CE4 上是没有默认路由的，原因是因为

![image-20211006194707998](https://i.loli.net/2021/10/06/CBFomqG2P7EItOe.png)

PE3

```sql
ospf 10
	default-route-advertise
```

PE4

```sql
ospf 20
	default-route-advertise
```



查看

CE3、CE4

```sql
<CE3>disp ip rou 0.0.0.0
Route Flags: R - relay, D - download to fib
------------------------------------------------------------------------------
Destination/Mask    Proto   Pre  Cost      Flags NextHop         Interface
        0.0.0.0/0   O_ASE   150  1           D   10.3.33.1       Mp-group0/0/0

<CE4>disp ip rou 0.0.0.0
Route Flags: R - relay, D - download to fib
------------------------------------------------------------------------------
Destination/Mask    Proto   Pre  Cost      Flags NextHop         Interface
        0.0.0.0/0   O_ASE   150  1           D   10.3.34.1       GigabitEthernet0/0/1
```

补充：

CE2 可以配置 BGP 传给 PE2

```sql
bgp 65000
	peer 10.2.22.5 default-route-advertise
```



####  4.2 NAT（2分）

1. 在CE1上，10.3.0.0/16（不含10.3.2.10）的内网地址转换为102.0.1.2-102.0.1.6，通过Ge2/0/0访问ISP。sever1拥有单独的公网地址 102.0.1.1，对ISP提供FTP和HTTP（2） 

解法： 注意：此处NAT映射的公网地址通常称为业务地址（102.0.1.2-6），与CE1和ISP接口互联地址（100.0.1.0/30）不一致 

CE1

```sql
undo int lo1
acl 2000  
	rule 5 deny source 10.3.2.10 0  
	rule 10 permit source 10.3.0.0 0.0.255.255  
  q
nat address-group 1 102.0.1.2 102.0.1.6 
interface GigabitEthernet2/0/0
	ip address 100.0.1.1 255.255.255.252  
	nat outbound 2000 address-group 1 
interface GigabitEthernet2/0/0 
	nat server protocol tcp global 102.0.1.1 ftp inside 10.3.2.10 ftp 
	nat server protocol tcp global 102.0.1.1 www inside 10.3.2.10 www
```

 验证： 在家练习的时候可以在ISP添加回包静态路由，访问ISP的8.8.8.8

ISP

```
 ip route-static 102.0.1.0 29 100.0.1.1 
```

考试ISP不能登录（虽然密码为hcie 一登就登上了），但是不用测试，保证NAT配置无误即可

pc1 

```sql
PC>ping 8.8.8.8
```

![image-20211006220647484](https://i.loli.net/2021/10/06/TQmtcCsLErWe3JM.png)

CE3

```sql
[CE3]ping -a 172.17.1.3 8.8.8.8
  PING 8.8.8.8: 56  data bytes, press CTRL_C to break
    Reply from 8.8.8.8: bytes=56 Sequence=1 ttl=248 time=110 ms
    Reply from 8.8.8.8: bytes=56 Sequence=2 ttl=248 time=100 ms
    Reply from 8.8.8.8: bytes=56 Sequence=3 ttl=248 time=110 ms
    Reply from 8.8.8.8: bytes=56 Sequence=4 ttl=248 time=110 ms
    Reply from 8.8.8.8: bytes=56 Sequence=5 ttl=248 time=130 ms
```

问题：为什么 CE1 上没有接口是 `102.0.1.0` 网段的，那为啥可以，

因为在配置 `	nat outbound 2000 address-group 1 `的时候，会自动生成用户自定义的路由条目

```sql
[CE1]disp ip routing
     102.0.1.1/32  Unr     64   0           D   127.0.0.1       InLoopBack0
     102.0.1.2/32  Unr     64   0           D   127.0.0.1       InLoopBack0
     102.0.1.3/32  Unr     64   0           D   127.0.0.1       InLoopBack0
     102.0.1.4/32  Unr     64   0           D   127.0.0.1       InLoopBack0
     102.0.1.5/32  Unr     64   0           D   127.0.0.1       InLoopBack0
     102.0.1.6/32  Unr     64   0           D   127.0.0.1 
```



####  4.3 QOS（7分）

1. 在CE1和G2/0/0的出方向，周一至周五的8：00-18：00点对TCP目的端口号6881-6999流量，承诺平均速率为1Mbps（3） 

##### 流量监管

CE1

定义一个时间

```sql
 time-range W 08:00 to 18:00 working-day 
```

在用 `高级 acl` 匹配流量

```sql
acl 3000  
 rule permit tcp destination-port range 6881 6999 time-range W 
```

CE1  的出方向 `g2/0/0` 进行限速

```sql
int g2/0/0
	qos car outbound acl 3000 cir 1000
```

这一步不需要验证

2. CE4-PE4的QOS规划如下表所示： 

![image-20211006164413483](https://i.loli.net/2021/10/06/oElWOGhP1ptH8iB.png)

​	在CE4的G0/0/1出方向对流量进行802.1p标记。在PE4的G0/0/1入方向，继承CE4 的802.1p值，并将802.1p映射为Dscp（2）

#####  分类标记

先复杂流（五元组） --- 简单流

![image-20211007104212544](https://i.loli.net/2021/10/07/vHQKbjeSmn3N8rk.png)



![image-20211007110307350](https://i.loli.net/2021/10/07/AXqWYv2E9wPZpsF.png)

###### 1. CE4 配置 --- 复杂流分类（MQC）

1. acl 区别业务网段

```sql
acl 3001  
	rule permit ip destination 10.3.1.0 0.0.0.255
acl 3002  
	rule permit ip destination 10.3.2.0 0.0.0.255
acl 3003  
	rule permit ip destination 10.3.3.0 0.0.0.255
acl 3004  
	rule permit ip destination 10.3.4.0 0.0.0.255
```

2. 创建复杂流分类匹配出流量

```sql
traffic classifier R
 	if-match acl 3001
traffic classifier S
 	if-match acl 3002
traffic classifier M
 	if-match acl 3003
traffic classifier O
 	if-match acl 3004
```

3. 创建流行为标记 `802.1p` 的值

```sql
traffic behavior R
	remark 8021p 5
traffic behavior S
	remark 8021p 4
traffic behavior M
	remark 8021p 3
traffic behavior O
	remark 8021p 2
```

4. 创建流策略进行关联流分类和流行为关系

```sql
traffic policy RM
	classifier R behavior R
	classifier S behavior S
	classifier M behavior M
	classifier O behavior O
```

5. 在 CE4 的 `g0/0/1` 出方向进行调用

```sql
int g0/0/1
	traffic-policy RM outbound
```

`注意:` 如果想要看到实验现象的话得需要**子接口**



###### 2. PE4 配置简单流分类

1. 将 `802.1p` 的值映射成 `dscp`

```sql
qos map-table dot1p-dscp  
  input 5 output 46
```

![image-20211007113959393](https://i.loli.net/2021/10/07/mJjauVs5oUnOTrP.png)

2. PE4 的入接口 `g0/0/1` 得信任 CE4 传来的 `802.1p`

```sql
int g0/0/1
	trust 8021p override
```



##### 拥塞管理和拥塞控制

队列技术、丢弃技术

![image-20211007115243700](https://i.loli.net/2021/10/07/816RQjFKlH2y7WL.png)

路由器会自动把 `dscp` 值 加入到不同的 `queue` 中，此时需要通过 `schedue` 技术来管理这些队列

设备上，每个接口出方向都拥有4个或8个队列，以队列索引号进行标识，队列索引号分别为0、1、2、3或0、1、2、3、4、5、6、7。设备根据本地优先级和队列之间的映射关系，自动将分类后的报文流送入各队列，然后按照各种队列调度机制进行调度。下面以每个接口8个队列对各种调度方式进行说明。

1. PQ

这种方式它严格按照队列优先级来，横向一个一个走，这样的缺点就是优先级低的队列容易被饿死

PQ调度，针对于关键业务类型应用设计，PQ调度算法维护一个优先级递减的队列系列并且只有当更高优先级的所有队列为空时才服务低优先级的队列。这样，将关键业务的分组放入较高优先级的队列，将非关键业务（如E-Mail）的分组放入较低优先级的队列，可以保证关键业务的分组被优先传送，非关键业务的分组在处理关键业务数据的空闲间隙被传送。

![PQ](https://i.loli.net/2021/10/07/vIsqrlLRCP5GcdE.gif)

1. WFQ

这种方式按照 `weight` 方式来，把流量分段，竖向的一个一个走

加权循环调度WRR（Weight Round Robin）在循环调度RR（Round Robin）的基础上演变而来，在队列之间进行轮流调度，根据每个队列的权重来调度各队列中的报文流。

![](https://i.loli.net/2021/10/07/webYjltdExX72oV.gif)

`TAIL` 尾丢弃 --- 大家一起死

`RED` 早期随机丢弃 --- 

`WRED`



##### 限速技术

流量监管、流量整形 --- 令牌桶算法 评估

3. PE4的G0/0/0和G0/0/2匹配DSCP值，根据表1，配置拥塞管理和拥塞避免。（2）

PE4

1. 创建 `WRED` 丢弃模板 `data` 

```sql
drop-profile data
  # 配置当前WRED丢弃模板基于DSCP优先级进行丢弃
	wred dscp 
    dscp cs4 low-limit 70 high-limit 100 discard-percentage 50
    dscp cs3 low-limit 50 high-limit 90 discard-percentage 50
    dscp cs2 low-limit 50 high-limit 80 discard-percentage 50
    dscp default low-limit 50 high-limit 80 discard-percentage 50
```

2. 创建队列模板 `queue-profile1`

```sql
qos queue-profile queue-profile1
	# 配置调度策略
  schedule pq 5
  schedule wfq 0 to 4
  # 配置 权重
	queue 4 weight 63
	queue 3 weight 21
	queue 2 weight 9
	queue 0 weight 1
	# 配置 wfq 丢弃模板
	queue 0 drop-profile data
  queue 2 to 4 drop-profile data
```

3. 在出接口下应用队列模板 `queue-profile1`

```sql
int g0/0/0
	qos queue-profile queue-profile1
int g0/0/2
	qos queue-profile queue-profile1
```



### 5. IPV6（14分）

####  5.1 基础配置

#### 5.2 IPV6 ISIS（3）

1. 如图6，PE1,PE2,RR1,P1,ASBR1,ASBR2,运行ISIS协议，各直接网段通告入ISIS，配置各链路cost（3） 

![image-20211007161442136](https://i.loli.net/2021/10/07/4sWpyRx6NcuXDdE.png)

注意：以上 6 台设备一定要打开 isis 的多拓扑能力

```sql
sy
isis
	ipv6 enable topology ipv6
```

以 PE1 为例配置

```sql
interface Ip-Trunk1
	isis ipv6 enable
 	isis ipv6 cost 1550
int g0/0/0
	isis ipv6 enable
 	isis ipv6 cost 20
int LoopBack0
	isis ipv6 enable
```

检查：

```sql
disp isis int  # 看接口 ipv6 up
disp ipv6 routing pro isis # 看到 D1 to D6 可以验证下 cost 值
```



#### 5.3 IPV6 BGP（11分）

1. 如图7，ASBR1-ASBR3通过直接链路建立EBGP4+邻居。PE1,PE2,P1是RR1的IBGP+客户站。（已配） 

![image-20211007173256195](https://i.loli.net/2021/10/07/SojPAfMO3GFieUw.png)

###### 1. ASBR1 、ASBR3 建立 EBGP4+ 需求

ASBR1 配置

先在 ASBR1、3 之间查看 `ipv6` 地址

```sql
bgp 100
	peer 2570:CCDD:CCBB:3CAF:EFFE:ACDD:CCDB:5700 as 200
	ipv6-family unicast
		peer 2570:CCDD:CCBB:3CAF:EFFE:ACDD:CCDB:5700 enable
```

ASBR3 配置

```sql
bgp 200
	peer 2570:CCDD:CCBB:3CAF:EFFE:ACDD:CCDB:5701 as 100
	ipv6-family unicast
		peer 2570:CCDD:CCBB:3CAF:EFFE:ACDD:CCDB:5701 enable
```

###### 2. PE1、PE2、P1 是 RR1 的 IBGP+

RR1 配置

```sql
bgp 100
	peer 2000:EAD8:99EF:C03E:B2AD:9EFF:32DD:DC01 as 100
	peer 2000:EAD8:99EF:C03E:B2AD:9EFF:32DD:DC01 con lo0
	peer 2000:EAD8:99EF:C03E:B2AD:9EFF:32DD:DC02 as 100
	peer 2000:EAD8:99EF:C03E:B2AD:9EFF:32DD:DC02 con lo0
	peer 2000:EAD8:99EF:C03E:B2AD:9EFF:32DD:DC04 as 100
	peer 2000:EAD8:99EF:C03E:B2AD:9EFF:32DD:DC04 con lo0
	peer 2000:EAD8:99EF:C03E:B2AD:9EFF:32DD:DC05 as 100
	peer 2000:EAD8:99EF:C03E:B2AD:9EFF:32DD:DC05 con lo0
	peer 2000:EAD8:99EF:C03E:B2AD:9EFF:32DD:DC06 as 100
	peer 2000:EAD8:99EF:C03E:B2AD:9EFF:32DD:DC06 con lo0
	
	ipv6-family unicast
  	peer 2000:EAD8:99EF:C03E:B2AD:9EFF:32DD:DC01 enable
		peer 2000:EAD8:99EF:C03E:B2AD:9EFF:32DD:DC01 reflect-client
		peer 2000:EAD8:99EF:C03E:B2AD:9EFF:32DD:DC02 enable
		peer 2000:EAD8:99EF:C03E:B2AD:9EFF:32DD:DC02 reflect-client
		peer 2000:EAD8:99EF:C03E:B2AD:9EFF:32DD:DC04 enable
		peer 2000:EAD8:99EF:C03E:B2AD:9EFF:32DD:DC04 reflect-client
		peer 2000:EAD8:99EF:C03E:B2AD:9EFF:32DD:DC05 enable
		peer 2000:EAD8:99EF:C03E:B2AD:9EFF:32DD:DC06 enable
```

PE1、PE2、P1、ASBR1、ASBR2 配置

```sql
bgp 100
	peer 2000:EAD8:99EF:C03E:B2AD:9EFF:32DD:DC03 as 100
	peer 2000:EAD8:99EF:C03E:B2AD:9EFF:32DD:DC03 con lo0
	ipv6-family unicast
  	peer 2000:EAD8:99EF:C03E:B2AD:9EFF:32DD:DC03 enable
```

验证：

```sql
# ASBR3 检查 EBGP4+
disp bgp ipv6 peer
# RR 检查 IBGP+  

disp bgp ipv6 peer
# 有 5 个邻居
# 发现有 DC01 - DC06 就行
```



2. 在ASBR1将ISIS IPV6的路由导入BGP4+,只向ASBR3通告前缀为2000:EAD8:99EF:C03E:B2AD:9EFF:32DD:DC00/120的路由（不能使用route-policy）。 将ASBR3的loopback0通告入BGP4+（4） 

> A、在ASBR1将ISIS IPV6的路由导入BGP4+

​	ASBR1 配置

```sql
bgp 100
	ipv6-family unicast
		import-route isis 1
```

​	ASBR3 查看 --- 能看到 `13` 条就完事了

```sql
[ASBR3]disp bgp ipv6 routing-table 

 BGP Local router ID is 10.1.78.1 
 Status codes: * - valid, > - best, d - damped,
               h - history,  i - internal, s - suppressed, S - Stale
               Origin : i - IGP, e - EGP, ? - incomplete
               
 Total Number of Routes: 13
```

> B、只向ASBR3通告前缀为2000:EAD8:99EF:C03E:B2AD:9EFF:32DD:DC00/120的路由（不能使用route-policy）

注意：`ipv6` 没有自动汇总，需手工配置 `aggregate`

ASBR1 配置

```sql
bgp 100
	ipv6-family unicast
  	aggregate 2000:EAD8:99EF:C03E:B2AD:9EFF:32DD:DC00 120 detail-suppressed # 抑制明细
```

此时 ASBR3 查看 就只有一条汇总路由了

```sql
[ASBR3]disp bgp ipv6 routing-table 

 BGP Local router ID is 10.1.78.1 
 Status codes: * - valid, > - best, d - damped,
               h - history,  i - internal, s - suppressed, S - Stale
               Origin : i - IGP, e - EGP, ? - incomplete

 Total Number of Routes: 1
 *>  Network  : 2000:EAD8:99EF:C03E:B2AD:9EFF:32DD:DC00  PrefixLen : 120    
     NextHop  : 2570:CCDD:CCBB:3CAF:EFFE:ACDD:CCDB:5701  LocPrf    :          
     MED      :                                          PrefVal   : 0         
     Label    : 
     Path/Ogn : 100  ?
```

是不是此时的现象很爽？是因为 `PE RR ASBR` 之间的 `ipv6` 互通地址前缀，正好在你的 `/120` 位前缀内，所以一并被汇总了

注意：考场直连网段可能不在我们的汇总的路由范围内，所以需要通过 `路由过滤` 实现只发送汇总路由

此时模拟 `ipv6`互通地址并不能正好被 `/120` 合并

PE1 和 PE2 的 `g0/0/0` 的 ipv6 地址改掉

PE1 

```sql
int g0/0/0
	undo ipv6 address 2000:EAD8:99EF:C03E:B2AD:9EFF:32DD:DC20/127
  ipv6 address 2012:: 127
```

PE2

```sql
int g0/0/0
	undo ipv6 address 2000:EAD8:99EF:C03E:B2AD:9EFF:32DD:DC21/127
  ipv6 address 2012::1 127
```

此时 ASBR3 查看 就会多出 `2012::` 前缀的路由

```sql
[ASBR3]disp bgp ipv6 routing-table
	Total Number of Routes: 2
 *>  Network  : 2000:EAD8:99EF:C03E:B2AD:9EFF:32DD:DC00  PrefixLen : 120    
     NextHop  : 2570:CCDD:CCBB:3CAF:EFFE:ACDD:CCDB:5701  LocPrf    :          
     MED      :                                          PrefVal   : 0         
     Label    : 
     Path/Ogn : 100  ?
 *>  Network  : 2012::                                   PrefixLen : 127    
     NextHop  : 2570:CCDD:CCBB:3CAF:EFFE:ACDD:CCDB:5701  LocPrf    :          
     MED      : 2430                                     PrefVal   : 0         
     Label    : 
     Path/Ogn : 100  ?
```

我们要做的事情呢就简单了，就是在 ASBR1 上把 `2012::` 前缀的 `过滤` 掉

> C、（不能使用route-policy）

ASBR1 配置

1. 先写一个 `ipv6 prefix` 匹配出 `DC00/120`

```sql
ip ipv6-prefix toASBR3 permit 2000:EAD8:99EF:C03E:B2AD:9EFF:32DD:DC00 120
```

2. 在  bgp `ipv6-family` 下对邻居 ASBR3 执行 `ipv6 prefix` 只往外发送汇总路由

```sql
bgp 100
	ipv6-family unicast
		peer 2570:CCDD:CCBB:3CAF:EFFE:ACDD:CCDB:5700 ipv6-prefix toASBR3 export
```

3. 此时 ASBR3 上查看 --- 此时就只有 汇总路由了

```sql
[ASBR3]disp bgp ipv6 routing-table 

 Total Number of Routes: 1
 *>  Network  : 2000:EAD8:99EF:C03E:B2AD:9EFF:32DD:DC00  PrefixLen : 120    
     NextHop  : 2570:CCDD:CCBB:3CAF:EFFE:ACDD:CCDB:5701  LocPrf    :          
     MED      :                                          PrefVal   : 0         
     Label    : 
     Path/Ogn : 100  ?
```



> D、将ASBR3的loopback0通告入BGP4+

ASBR3 配置

```sql
bgp 200
	ipv6-family unicast
		network 2000:EAD8:99EF:C03E:B2AD:9EFF:32DD:DC07 128 
```



2. PE1,PE2学习到ASBR3 loopback0的BGP4+明细路由（3） 

PE1 上查看 `ipv6 routing` --- 发现只有自己的链路本地地址

```sql
[PE1]disp bgp ipv6 routing-table 
 Total Number of Routes: 2
   i Network  : 2000:EAD8:99EF:C03E:B2AD:9EFF:32DD:DC00  PrefixLen : 120    
     NextHop  : 2000:EAD8:99EF:C03E:B2AD:9EFF:32DD:DC05  LocPrf    : 100       
     MED      :                                          PrefVal   : 0         
     Label    : 
     Path/Ogn : ?
   i Network  : 2012::                                   PrefixLen : 127    
     NextHop  : 2000:EAD8:99EF:C03E:B2AD:9EFF:32DD:DC05  LocPrf    : 100       
     MED      : 2430                                     PrefVal   : 0         
     Label    : 
     Path/Ogn : ?
```

 解法：RR1 和 P1 部署 `ipv6` 路由渗透

PP1、P1

1. 写一个 `ipv6 prefix` 把所有的环回口匹配出来

```sql
ip ipv6-prefix LOOP permit 2000:EAD8:99EF:C03E:B2AD:9EFF:32DD:DC00 120 greater-equal 128 less-equal 128 # 正确
ip ipv6-prefix LOOP permit 2000:EAD8:99EF:C03E:B2AD:9EFF:32DD:DC 120 greater-equal 128 less-equal 128 # 错误
```

2. 路由渗透时候调用 `filter-policy`

```sql
isis
	ipv6 import-route isis level-2 into level-1 filter-policy ipv6-prefix LOOP
```

3. PE1、PE2 上查看 --- 发现没有 ASBR3 的明细路由 `DC07`，只有一条汇总路由 + `2012::`

```sql
[PE1]disp bgp ipv6 routing-table

 Total Number of Routes: 2
 *>i Network  : 2000:EAD8:99EF:C03E:B2AD:9EFF:32DD:DC00  PrefixLen : 120    
     NextHop  : 2000:EAD8:99EF:C03E:B2AD:9EFF:32DD:DC05  LocPrf    : 100       
     MED      :                                          PrefVal   : 0         
     Label    : 
     Path/Ogn : ?
 *>i Network  : 2012::                                   PrefixLen : 127    
     NextHop  : 2000:EAD8:99EF:C03E:B2AD:9EFF:32DD:DC05  LocPrf    : 100       
     MED      : 2430                                     PrefVal   : 0         
     Label    : 
     Path/Ogn : ?
```

4. 在 RR1、P1 上查看也是如此 ---  这就说明 ·ASBR3 的明细正好被 ASBR1 的路由汇总汇总了

注意：考场可能出现ASBR3的环回口路由在汇总路由范围内，也被ASBR1抑制明细，需要配置ASBR1不抑制ASBR3的明细路由

解法：ASBR1 写一个 `ipv6 prefix` 把 `DC07` 匹配出来，在写一个 `route-policy` deny `DC07`，放行其它

​				然后汇总的时候调用 `suppress-policy` 完事了 

A、写一个 `ipv6 prefix` 把 `DC07` 匹配出来

```sql
ip ipv6-prefix ASBR3 permit 2000:EAD8:99EF:C03E:B2AD:9EFF:32DD:DC07 128
```

B、写一个 `route-policy`

```sql
route-policy SUPPRESS deny node 10
	if-match ipv6 address prefix-list ASBR3
route-policy SUPPRESS permit node 20 
```

C、`aggregate` 时加上 `suppress-policy`

```sql
bgp 100
	ipv6-family unicast
  	aggregate 2000:EAD8:99EF:C03E:B2AD:9EFF:32DD:DC00 120 detail-suppressed suppress-policy SUPPRESS
```

D、P1、RR1 上查看 --- 发现 `DC07` 没有最优

```sql
[RR1]disp bgp ipv6 routing-table 

 Total Number of Routes: 3
 *>i Network  : 2000:EAD8:99EF:C03E:B2AD:9EFF:32DD:DC00  PrefixLen : 120    
     NextHop  : 2000:EAD8:99EF:C03E:B2AD:9EFF:32DD:DC05  LocPrf    : 100       
     MED      :                                          PrefVal   : 0         
     Label    : 
     Path/Ogn : ?
   i Network  : 2000:EAD8:99EF:C03E:B2AD:9EFF:32DD:DC07  PrefixLen : 128    
     NextHop  : 2570:CCDD:CCBB:3CAF:EFFE:ACDD:CCDB:5700  LocPrf    : 100       
     MED      : 0                                        PrefVal   : 0         
     Label    : 
     Path/Ogn : 200  i
 *>i Network  : 2012::                                   PrefixLen : 127    
     NextHop  : 2000:EAD8:99EF:C03E:B2AD:9EFF:32DD:DC05  LocPrf    : 100       
     MED      : 2430                                     PrefVal   : 0         
     Label    : 
     Path/Ogn : ?
```

这是怎么回事呢？啊原来是 ASBR1 从 EBGP 学来的路由 通过 IBGP 传递给 RR 时候，要设置 `next-hop-local`

E、ASBR1 配置 `next-hop-local`

```sql
 bgp 100
	ipv6-family unicast
 		peer 2000:EAD8:99EF:C03E:B2AD:9EFF:32DD:DC03 next-hop-local
```

F、PE1、PE2 上查看 -- 发现有汇总也有 ASBR3 的明细路由了

```sql
[PE1]disp bgp ipv6 routing-table

 Total Number of Routes: 3
 *>i Network  : 2000:EAD8:99EF:C03E:B2AD:9EFF:32DD:DC00  PrefixLen : 120    
     NextHop  : 2000:EAD8:99EF:C03E:B2AD:9EFF:32DD:DC05  LocPrf    : 100       
     MED      :                                          PrefVal   : 0         
     Label    : 
     Path/Ogn : ?
 *>i Network  : 2000:EAD8:99EF:C03E:B2AD:9EFF:32DD:DC07  PrefixLen : 128    
     NextHop  : 2000:EAD8:99EF:C03E:B2AD:9EFF:32DD:DC05  LocPrf    : 100       
     MED      : 0                                        PrefVal   : 0         
     Label    : 
     Path/Ogn : 200  i
 *>i Network  : 2012::                                   PrefixLen : 127    
     NextHop  : 2000:EAD8:99EF:C03E:B2AD:9EFF:32DD:DC05  LocPrf    : 100       
     MED      : 2430                                     PrefVal   : 0         
     Label    : 
     Path/Ogn : ?
```

PE1 访问 ASBR3

```sql
ping ipv6 -a 2000:EAD8:99EF:C03E:B2AD:9EFF:32DD:DC01 2000:EAD8:99EF:C03E:B2AD:9EFF:32DD:DC07
 PING 2000:EAD8:99EF:C03E:B2AD:9EFF:32DD:DC07 : 56  data bytes, press CTRL_C to
 break
    Reply from 2000:EAD8:99EF:C03E:B2AD:9EFF:32DD:DC07 
    bytes=56 Sequence=1 hop limit=62  time = 70 ms
    Reply from 2000:EAD8:99EF:C03E:B2AD:9EFF:32DD:DC07 
    bytes=56 Sequence=2 hop limit=62  time = 100 ms
    Reply from 2000:EAD8:99EF:C03E:B2AD:9EFF:32DD:DC07 
    bytes=56 Sequence=3 hop limit=62  time = 80 ms
    Reply from 2000:EAD8:99EF:C03E:B2AD:9EFF:32DD:DC07 
    bytes=56 Sequence=4 hop limit=62  time = 80 ms
    Reply from 2000:EAD8:99EF:C03E:B2AD:9EFF:32DD:DC07 
    bytes=56 Sequence=5 hop limit=62  time = 80 ms
```




3. 请在PE1使能某特性，以确保PE1在启动过程（从物理接口UP）到各协议邻居建立，PE2-ASBR3的IPV6 ping无丢包（4）

​	PE1 配置

A、IGP 与 BGP 联动

```sql
isis
	set-overload on-startup wait-for-bgp
```

B、IGP 与 IDP 联动

```sql
int Ip-Trunk1
	isis ldp-sync
int g0/0/0
	isis ldp-sync
```



下一周任务

LAB1 - LAB2 敲熟

论述题

TS：排错

TAC：诊断



```flow
st=>start: 闹钟响起
op=>operation: 与床板分离
cond=>condition: 分离成功?
e=>end: 快乐的一天

st->op->cond
cond(yes)->e
cond(no)->op
```

