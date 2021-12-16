### TS3.0 排错

#### MPLS-VPN实现site1-site4互访

 Site1与Site4为同一个VPN客户的两个站点，现在site1里的CLIENTS无法和site4里的CLIENT通信，请解决此问题； 
$\textcolor{red}{注意：不要删除现有配置，可修改解决}$

注意： 目前考场环境 Option-A 是直接将物理接口绑定到对应VPN实例 （VRF-to-VRF），所以该接口无法实现BGP IPv4公网、BGP IPv6 公网互访要求，NAT 需求也只能访问 AS200 分析

考察 OPTION A

##### 可能故障原因：

1. 控制层面传递路由故障
   1. PE设备(R23) ISIS-BGP 双向引入错误 --- R1 已经在选路的时候解决了
   2. VPNV4 对等体建立错误，ASBR 之间未建立基于 VPN 实例对等体
   3. RR 路由反射器客户端配置错误
   4. RR 未关闭 `RT`值检测
   5. PE 设备和本端 ASBR  `RT` 值检测未开及 `RT` 值配置错误
2. 转发层面 --- 底层 LSP 没有成功建立
   1. 底层 IGP 存在问题
   2. LDP 会话没有建立
3. ASBR 之间接口`不需要`开启` MPLS` 能力

##### 解决方案

###### 1. 检查AR23的BGP VPNv4路由----双向引入是否正确

```sql
[AR23]disp bgp vpnv4 all peer # 邻居有
  Peer            V          AS  MsgRcvd  MsgSent  OutQ  Up/Down       State Pre
fRcv
  200.1.1.9       4         200      608      591     0 09:38:45 Established    
   0
[AR23]disp bgp vpnv4 all routing  # 空的
[AR23]disp ip rou vpn-instance 1 # vpn-instance 1
Route Flags: R - relay, D - download to fib
------------------------------------------------------------------------------
Routing Tables: 1
         Destinations : 10       Routes : 13       

Destination/Mask    Proto   Pre  Cost      Flags NextHop         Interface

       10.4.1.0/24  ISIS-L2 15   10          D   10.4.128.24     GigabitEthernet2/0/0
      10.4.1.23/32  Direct  0    0           D   127.0.0.1       LoopBack1
      10.4.1.25/32  ISIS-L2 15   10          D   10.4.128.25     GigabitEthernet2/0/0
      10.4.1.26/32  ISIS-L2 15   20          D   10.4.128.24     GigabitEthernet2/0/0
                    ISIS-L2 15   20          D   10.4.128.25     GigabitEthernet2/0/0
      10.4.27.0/24  ISIS-L2 15   30          D   10.4.128.24     GigabitEthernet2/0/0
                    ISIS-L2 15   30          D   10.4.128.25     GigabitEthernet2/0/0
     10.4.128.0/24  Direct  0    0           D   10.4.128.23     GigabitEthernet2/0/0
    10.4.128.23/32  Direct  0    0           D   127.0.0.1       GigabitEthernet2/0/0
   10.4.128.255/32  Direct  0    0           D   127.0.0.1       GigabitEthernet2/0/0
     10.4.129.0/24  ISIS-L2 15   20          D   10.4.128.24     GigabitEthernet2/0/0
                    ISIS-L2 15   20          D   10.4.128.25     GigabitEthernet2/0/0
255.255.255.255/32  Direct  0    0           D   127.0.0.1       InLoopBack0
```

bgp vpnv4 没有，vpn-instance 1 发现有 Site4 PC 网段地址 `10.4.27.0/24`，这是怎么回事呢？

```sql
bgp 200
	ipv4-family vpn-instance 1 
  	import-route isis 100 route-policy I2B
```

看一下 route-policy I2B

```sql
route-policy I2B permit node 10 
	if-match acl 2001 
```

接着看 acl 2001

```sql
acl 2001  
	rule 1000 deny
```

好家伙，真绝啊，全 `deny` 了，在不删除配置的情况下，增加规则

```sql
acl 2001  
	rule 10 permit source 10.4.0.0 0.0.255.255 
  rule 1000 deny 
```

接着查看一下 vpnv4 路由，发现有了

```sql
[AR23]disp bgp vpnv4 all routing


 BGP Local router ID is 200.1.1.23 
 Status codes: * - valid, > - best, d - damped,
               h - history,  i - internal, s - suppressed, S - Stale
               Origin : i - IGP, e - EGP, ? - incomplete

 Total number of routes from all PE: 7
 Route Distinguisher: 200:100 

      Network            NextHop        MED        LocPrf    PrefVal Path/Ogn

 *>   10.4.1.0/24        0.0.0.0         10                    0      ?
 *>   10.4.1.23/32       0.0.0.0         0                     0      ?
 *>   10.4.1.25/32       0.0.0.0         10                    0      ?
 *>   10.4.1.26/32       0.0.0.0         20                    0      ?
 *>   10.4.27.0/24       0.0.0.0         30                    0      ?
 *>   10.4.128.0/24      0.0.0.0         0                     0      ?
 *>   10.4.129.0/24      0.0.0.0         20                    0      ?

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

B2I

```sql
isis 100 vpn-instance 1
	is-level level-2
  network-entity 47.0004.0000.0000.0023.00
  import-route bgp  # 加上此行
```

###### 2. 检查所有路由的 VPNV4 邻居关系

注意 AR2 和 AR4、AR2 和 AR5 之间基于 VPN 实例建立 EBGP 对等体

R9(23、4、5)、R2（4、5、7）、R7 （1、2、6、13）检查

```sql
disp bgp vpnv4 all peer 
```

R2 和 R4、R5 邻居 down

> R2 在 vpn-instance 1 下建立对等体

```sql
ipv4-family vpn-instance 1 
  peer 200.100.24.4 as-number 200 
  peer 200.100.25.5 as-number 200 
```

> 接口绑定实例

```sql
interface Pos5/0/0
  ip binding vpn-instance 1
  ip address 200.100.24.2 255.255.255.0 
interface GigabitEthernet2/0/0
  ip binding vpn-instance 1
  ip address 200.100.25.2 255.255.255.0
```

R4 配置

> vpn-instance 1 邻居

```sql
bgp 200 
  ipv4-family vpnv4
    policy vpn-target
    peer 200.1.1.9 enable
  ipv4-family vpn-instance 1 
    peer 200.100.24.2 as-number 100 
```

> 接口绑定实例

```sql
[AR4]disp ip vpn-instance 1 int
 VPN-Instance Name and ID : 1, 1
  Interface Number : 1 
  Interface list : Pos5/0/0
```



R5 配置

> vpn-instance 1 邻居

```sql
bgp 200
  ipv4-family vpnv4
    policy vpn-target
    peer 200.1.1.9 enable
  ipv4-family vpn-instance 1 
    peer 200.100.25.2 as-number 100 
```

> 接口绑定实例

```sql
[AR5]disp ip vpn-instance 1 int
 VPN-Instance Name and ID : 1, 1
  Interface Number : 1 
  Interface list : GigabitEthernet2/0/0
```



验证：

R2 查看邻居

```sql
<AR2>disp bgp vpnv4 vpn-instance 1 peer

 BGP local router ID : 100.1.1.2
 Local AS number : 100

 VPN-Instance 1, Router ID 100.1.1.2:
 Total number of peers : 2		  Peers in established state : 2

  Peer            V          AS  MsgRcvd  MsgSent  OutQ  Up/Down       State PrefRcv
  200.100.24.4    4         200        3        4     0 00:01:15 Established    0
  200.100.25.5    4         200        2        3     0 00:00:04 Established    

```



###### 3. R7 和 R9 作为 VPNv4 路由反射器，配置正确客户端

R7 配置

```sql
bgp 100
ipv4-family vpnv4
  peer 100.1.1.1 reflect-client
  peer 100.1.1.2 reflect-client
```

R9 配置

```sql
bgp 200
ipv4-family vpnv4
  peer 200.1.1.4 reflect-client
  peer 200.1.1.5 reflect-client
  peer 200.1.1.23 relect-client
```

###### 4. RR(R7、R9) 关闭 RT 检测、PE(R1、R23)，ASBR(R2、R4、R5)  开启 RT 值检测

 R7、AR9

```sql
ipv4-family vpnv4
	undo policy vpn-target
```

R1、R23、R2、R4、R5

```SQL
ipv4-family vpnv4
  policy vpn-target
```

![image-20211020193427578](https://i.loli.net/2021/10/20/8jOkIeqvctwlNr4.png)

###### 5. PE 与本端 ASBR 的 `RT` 配置错误

考场可能在 R1 或者 R2 错误配置 `RT` 值

R1 修改 RT 值 --- 不删原来的，加一下就行了

```sql
ip vpn-instance 1
  vpn-target 200:100 both 
```

R23、R9、R4、R5 

`disp bgp vpnv4 all routing-table`

R4 和 R5 不是最优的

```sql
<AR4>disp bgp vpnv4 all routing-table 


 BGP Local router ID is 200.1.1.4 
 Status codes: * - valid, > - best, d - damped,
               h - history,  i - internal, s - suppressed, S - Stale
               Origin : i - IGP, e - EGP, ? - incomplete

 Total number of routes from all PE: 7
 Route Distinguisher: 200:100 


      Network            NextHop        MED        LocPrf    PrefVal Path/Ogn

 *>i  10.4.1.0/24        200.1.1.23      10         100        0      ?
 *>i  10.4.1.23/32       200.1.1.23      0          100        0      ?
 *>i  10.4.1.25/32       200.1.1.23      10         100        0      ?
 *>i  10.4.1.26/32       200.1.1.23      20         100        0      ?
 *>i  10.4.27.0/24       200.1.1.23      30         100        0      ?
 *>i  10.4.128.0/24      200.1.1.23      0          100        0      ?
 *>i  10.4.129.0/24      200.1.1.23      20         100        0      ?

 VPN-Instance 1, Router ID 200.1.1.4:

 Total Number of Routes: 7
      Network            NextHop        MED        LocPrf    PrefVal Path/Ogn

   i  10.4.1.0/24        200.1.1.23      10         100        0      ?
   i  10.4.1.23/32       200.1.1.23      0          100        0      ?
   i  10.4.1.25/32       200.1.1.23      10         100        0      ?
   i  10.4.1.26/32       200.1.1.23      20         100        0      ?
   i  10.4.27.0/24       200.1.1.23      30         100        0      ?
   i  10.4.128.0/24      200.1.1.23      0          100        0      ?
   i  10.4.129.0/24      200.1.1.23      20         100        0      ?

```

看一下 ldp lsp 有没有建立好，``没有分配``去往 `200.1.1.23`、`200.1.1.9` 的 lsp

```sql
<AR4>disp mpls lsp
-------------------------------------------------------------------------------
                 LSP Information: LDP LSP
-------------------------------------------------------------------------------
FEC                In/Out Label  In/Out IF                      Vrf Name       
200.1.1.4/32       3/NULL        -/-                                           
200.1.1.5/32       NULL/3        -/GE0/0/0                                     
200.1.1.5/32       1024/3        -/GE0/0/0    
```

###### 6. 部分设备 LDP 会话未建立

看一下会话有没有建立，和 R9 没有建立好

```sql
<AR4>disp mpls ldp session all
 ------------------------------------------------------------------------------
 PeerID             Status      LAM  SsnRole  SsnAge      KASent/Rcv
 ------------------------------------------------------------------------------
 200.1.1.99:0       NonExistent      Passive              0/0
 200.1.1.5:0        Operational DU   Passive  0000:03:24  817/817
 ------------------------------------------------------------------------------
 TOTAL: 2 session(s) Found.
```

R4 ping 一下 R9，发现不通，说明 R4 或者 R9 底层不可达

```sql
<AR4>ping 200.1.1.99
  PING 200.1.1.99: 56  data bytes, press CTRL_C to break
    Request time out
    Request time out
    Request time out
    Request time out
```

R9

```sql
int loop1
	isis enable 200
```

在检查一下 ldp 会话

```sql
<AR4>disp mpls ldp session all
 ------------------------------------------------------------------------------
 PeerID             Status      LAM  SsnRole  SsnAge      KASent/Rcv
 ------------------------------------------------------------------------------
 200.1.1.99:0       Operational DU   Passive  0000:00:00  4/4
 200.1.1.5:0        Operational DU   Passive  0000:03:25  823/823
 ------------------------------------------------------------------------------
 TOTAL: 2 session(s) Found.
```

mpls lsp --- 都有了

```sql
<AR4>disp mpls lsp
-------------------------------------------------------------------------------
                 LSP Information: LDP LSP
-------------------------------------------------------------------------------
FEC                In/Out Label  In/Out IF                      Vrf Name       
200.1.1.4/32       3/NULL        -/-                                           
200.1.1.5/32       NULL/3        -/GE0/0/0                                     
200.1.1.5/32       1024/3        -/GE0/0/0                                     
200.1.1.99/32      NULL/3        -/GE0/0/1                                     
200.1.1.99/32      1028/3        -/GE0/0/1                                     
200.1.1.9/32       NULL/3        -/GE0/0/1                                     
200.1.1.9/32       1029/3        -/GE0/0/1                                     
200.1.1.23/32      NULL/1029     -/GE0/0/1                                     
200.1.1.23/32      1030/1029     -/GE0/0/1    
```

R4 查看 vpn-instance 路由，都是最优的

```sql
<AR4>disp bgp vpnv4 all routing-table 


 BGP Local router ID is 200.1.1.4 
 Status codes: * - valid, > - best, d - damped,
               h - history,  i - internal, s - suppressed, S - Stale
               Origin : i - IGP, e - EGP, ? - incomplete
 Total number of routes from all PE: 7
 Route Distinguisher: 200:100 
      Network            NextHop        MED        LocPrf    PrefVal Path/Ogn

 *>i  10.4.1.0/24        200.1.1.23      10         100        0      ?
 *>i  10.4.1.23/32       200.1.1.23      0          100        0      ?
 *>i  10.4.1.25/32       200.1.1.23      10         100        0      ?
 *>i  10.4.1.26/32       200.1.1.23      20         100        0      ?
 *>i  10.4.27.0/24       200.1.1.23      30         100        0      ?
 *>i  10.4.128.0/24      200.1.1.23      0          100        0      ?
 *>i  10.4.129.0/24      200.1.1.23      20         100        0      ?

 VPN-Instance 1, Router ID 200.1.1.4:

 Total Number of Routes: 7
      Network            NextHop        MED        LocPrf    PrefVal Path/Ogn
 *>i  10.4.1.0/24        200.1.1.23      10         100        0      ?
 *>i  10.4.1.23/32       200.1.1.23      0          100        0      ?
 *>i  10.4.1.25/32       200.1.1.23      10         100        0      ?
 *>i  10.4.1.26/32       200.1.1.23      20         100        0      ?
 *>i  10.4.27.0/24       200.1.1.23      30         100        0      ?
 *>i  10.4.128.0/24      200.1.1.23      0          100        0      ?
 *>i  10.4.129.0/24      200.1.1.23      20         100        0      ?

```

------

R2、R7 查看 -- 也都有了

R1 查看没有 `10.4.27.0/24` 

发现是 R1、R2 `RT` 值错误

R2

```sql
ip vpn-instance 1
	ipv4-family
  	route-distinguisher 200:100
  	vpn-target 200:100 export-extcommunity
  	vpn-target 200:100 import-extcommunity
```

R1

```sql
ip vpn-instance 1
 description site1-site4
 service-id 14
 ipv4-family
  	route-distinguisher 200:100
  	vpn-target 200:10 export-extcommunity  # 这里
  	vpn-target 200:10 import-extcommunity  # 这里
```

R1 改正

```sql
vpn-target 200:100 both
```

R1 查看路由 --- 发现都有了

```sql
[AR1]disp bgp vpnv4 all routing-table 


 BGP Local router ID is 100.1.1.1 
 Status codes: * - valid, > - best, d - damped,
               h - history,  i - internal, s - suppressed, S - Stale
               Origin : i - IGP, e - EGP, ? - incomplete



 Total number of routes from all PE: 16
 Route Distinguisher: 200:100 


      Network            NextHop        MED        LocPrf    PrefVal Path/Ogn

 *>   10.1.1.1/32        0.0.0.0         0                     0      ?
 *>   10.1.12.0/24       10.1.100.100    0                     0      300i
 *                       10.1.200.200    100                   0      300i
 *>   10.1.34.0/24       10.1.200.200    0                     0      300i
 *                       10.1.100.100    100                   0      300i
 *>   10.1.100.0/24      0.0.0.0         0                     0      ?
 *>   10.1.100.1/32      0.0.0.0         0                     0      ?
 *>   10.1.200.0/24      0.0.0.0         0                     0      ?
 *>   10.1.200.1/32      0.0.0.0         0                     0      ?
 *>i  10.4.1.0/24        100.1.1.2                  100        0      200?
 *>i  10.4.1.23/32       100.1.1.2                  100        0      200?
 *>i  10.4.1.25/32       100.1.1.2                  100        0      200?
 *>i  10.4.1.26/32       100.1.1.2                  100        0      200?
 *>i  10.4.27.0/24       100.1.1.2                  100        0      200?
 *>i  10.4.128.0/24      100.1.1.2                  100        0      200?
 *>i  10.4.129.0/24      100.1.1.2                  100        0      200?

 VPN-Instance 1, Router ID 100.1.1.1:

 Total Number of Routes: 16
      Network            NextHop        MED        LocPrf    PrefVal Path/Ogn

 *>   10.1.1.1/32        0.0.0.0         0                     0      ?
 *>   10.1.12.0/24       10.1.100.100    0                     0      300i
 *                       10.1.200.200    100                   0      300i
 *>   10.1.34.0/24       10.1.200.200    0                     0      300i
 *                       10.1.100.100    100                   0      300i
 *>   10.1.100.0/24      0.0.0.0         0                     0      ?
 *>   10.1.100.1/32      0.0.0.0         0                     0      ?
 *>   10.1.200.0/24      0.0.0.0         0                     0      ?
 *>   10.1.200.1/32      0.0.0.0         0                     0      ?
 *>i  10.4.1.0/24        100.1.1.2                  100        0      200?
 *>i  10.4.1.23/32       100.1.1.2                  100        0      200?
 *>i  10.4.1.25/32       100.1.1.2                  100        0      200?
 *>i  10.4.1.26/32       100.1.1.2                  100        0      200?
 *>i  10.4.27.0/24       100.1.1.2                  100        0      200?
 *>i  10.4.128.0/24      100.1.1.2                  100        0      200?
 *>i  10.4.129.0/24      100.1.1.2                  100        0      200?

```

7. Site4 client 14 --- Site2 client1、client3

```sql
PC>ping 10.1.12.11

Ping 10.1.12.11: 32 data bytes, Press Ctrl_C to break
From 10.1.12.11: bytes=32 seq=1 ttl=120 time=266 ms
From 10.1.12.11: bytes=32 seq=2 ttl=120 time=156 ms
From 10.1.12.11: bytes=32 seq=3 ttl=120 time=141 ms
From 10.1.12.11: bytes=32 seq=4 ttl=120 time=125 ms
From 10.1.12.11: bytes=32 seq=5 ttl=120 time=140 ms

PC>ping 10.1.34.33
Ping 10.1.34.33: 32 data bytes, Press Ctrl_C to break
Request timeout!
Request timeout!
Request timeout!
Request timeout!
Request timeout!

```

怎么 client1 通、client3 通呢？

R23 查看 vpnv4 路由

```sql
[AR23]disp bgp vpnv4 all routing-table 
 Route Distinguisher: 200:100 

      Network            NextHop        MED        LocPrf    PrefVal Path/Ogn

 *>i  10.1.1.1/32        200.1.1.4                  100        0      100?
 *>i  10.1.12.0/24       200.1.1.4                  100        0      100 300i  # 这里都有
 *>i  10.1.34.0/24       200.1.1.4                  100        0      100 300i  # 这里都有
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

 *>i  10.1.12.0/24       200.1.1.4                  100        0      100 300i # 有12，没有34
 *>   10.4.1.0/24        0.0.0.0         10                    0      ?
 *>   10.4.1.23/32       0.0.0.0         0                     0      ?
 *>   10.4.1.25/32       0.0.0.0         10                    0      ?
 *>   10.4.1.26/32       0.0.0.0         20                    0      ?
 *>   10.4.27.0/24       0.0.0.0         30                    0      ?
 *>   10.4.128.0/24      0.0.0.0         0                     0      ?
 *>   10.4.129.0/24      0.0.0.0         20                    0      ?
[AR23]
```

虽然 `Route Distinguisher: 200:100` 有，但是  `VPN-Instance 1` 并没有

```sql
[AR23]disp ip rou vpn-instance 1
Route Flags: R - relay, D - download to fib
------------------------------------------------------------------------------
Routing Tables: 1
         Destinations : 11       Routes : 14       

Destination/Mask    Proto   Pre  Cost      Flags NextHop         Interface

      10.1.12.0/24  IBGP    255  0          RD   200.1.1.4       GigabitEthernet
0/0/0
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
acl 2000  
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
> 8. tracert 看一下，相比较 OPTION B 没有断层

![image-20211020201251786](https://i.loli.net/2021/10/20/H7ZtBYLqeuxMzAb.png)

![image-20211020201438510](https://i.loli.net/2021/10/20/c2JR5SyQ9O6BIAm.png)

