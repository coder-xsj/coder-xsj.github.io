## LAB-2

二层 + IGP 和 `lab1` 一模一样

IGP --- 有可能这题顺序不一样，有可能放在MPLS VPN，解法一致

​	在RR2，P2上，ISIS和OSPF双向引入前缀为172.16.0.0/16的主机路由，被引入协议的cost要继承到后引入的协议中，P2和PE4的loopback0互 访走最优路径。配置要求有最好的扩展性。（8）



#### 3. MPLS VPN （45分）

1. CE1,CE2为VPN1的Hub-CE，PE1,PE2为Hub-PE；CE3，CE4为VPN的spoke站点；PE3，PE4为SPOKE-PE
2. CE4为Multi-VPN-instance CE1，CE4的VPN实例1，通过Ge0/0/1连接PE4。
3. 合理设置VPN1参数，使得Spoke站点互访的流量必须经过Hub-CE设备。当CE1-PE1链路断开的情况下，PE1仍然可以学习到CE1的业务路
    由。（PE3上的VPN1的RD为100:13,EXPORT RT为100：1，import RT为200：1）（2）
4. 如图4，CE1通过G0/0/1.1和G0/0/1.2建立直接EBGP邻居，接入PE1。CE1通过G0/0/1.2,向PE1通告BGP update中，某些路由信息的AS-path中
    有200。在CE1上，将OSPF路由导入BGP。（2）--------- `hub-spoke allow-as-loop （PE1、PE2 TOS , RR2 vpnv4）`
5. CE2通过G0/0/1.1和G0/0/1.2建立直接EBGP邻居，接入PE2。CE2通过G0/0/1.2,向PE2通告BGP update中，某些路由信息的AS-path中有200。
    在CE2上，将OSPF导入BGP。（2）
6. CE3通过OSPF区域1接入PE3，通过PE3-CE3的逻辑接口互通，通告CE3的各环回口；CE4通过OSPF区域0接入PE4，通过PE4-CE4的Ge0/0/1接口
    互通，通告CE4的各环回口；（2）
7. 如图4在AS100，AS200内建立IBGP IPV4邻居关系，RR1是PE1,PE2,P1,ASBR1,ASBR2的反射器，RR2是PE3，PE4，P2，ASBR4的反射器。ASBR1-ASBR3，ASBR2-ASBR4建立EBGP IPV4邻居关系------------`LAB-2没有BGP预配`
   
8. 如图3，AS100，AS200内各网元配置MPLS LSR-ID， 全局使能MPLS , MPLS LDP（已配）AS100,AS200内各有直连链路建立LDP邻居（除PE1-RR1之间，其余已配）（1）-------------------------------------------------------------以上需求和LAB1一致，解法一致！！！！
   分析：AS 100的ISIS部署存在路由泄露问题，需要手动将level 2的路由泄露进level 1
9. ASBR1-ASBR3、ASBR2-ASBR4之间通过直连接口建立EBGP邻居关系。在ASBR上将ISIS的loopback0口引入BGP。
   假设loopback0地址为172.16.1.Y/32，当Y为奇数时，对端设备访问本AS设备的loopback0优选的链路为ASBR1-ASBR3，
   当Y为偶数时，对端设备访问本AS设备的loopback0优选的链路为ASBR2-ASBR4，保证配置具有最好的扩展性。（10分）

10. 如图4，各站点，通过MPLS BGP VPN跨域OPTION C方案二，能够相互学习路由，MPLS域不能出现次优路径。（15）
11. CE1-PE1之间链路开，CE1设备仍可以学习到spoke业务网段。配置保障有最好的扩展性。（6）
12. 在拓扑正常情况下，要求CE1，CE2访问spoke网段时，不从本AS内绕行。（1）
13. 在PE3，PE4上修改BGP local-preference属性，实现CE3,CE4访问非直接的10.3.x.0/24网段时，若X为奇数，PE3，PE4优选的下一跳为PE1；若X为偶数，PE3，PE4优选的下一跳为PE2，不用考虑来回路径是否一致。（3分）-------------------和LAB1一致！！！！！

#### 1. 配置 4、7

**LAB2 没有 BGP 预配**

LAB1 还原成 LAB2

PE、RR、P、ASBR 全部删除 `bgp` 配置

```sql
sy
undo bgp 100
```

配置 `TOH TOS` 以及 `allow-as-loop`

PE1

```sql
bgp 100
	ipv4-family vpn-instance TOH
 		peer 10.2.11.2 as-number 65000
 	ipv4-family vpn-instance TOS
  	peer 10.2.11.6 as-number 65000
  	peer 10.2.11.6 allow-as-loop
```

PE2

```sql
bgp 100
	ipv4-family vpn-instance TOH
 		peer 10.2.22.2 as-number 65000
 	ipv4-family vpn-instance TOS
  	peer 10.2.22.6 as-number 65000
  	peer 10.2.22.6 allow-as-loop
```

##### 1. 配置 IBGP

RR1 配置

```sql
bgp 100
  peer 172.16.1.1 as 100
  peer 172.16.1.1 con lo0
  peer 172.16.1.1 reflect-client
  peer 172.16.1.20 as 100
  peer 172.16.1.20 con lo0
  peer 172.16.1.20 reflect-client

  peer 172.16.1.4 as 100
  peer 172.16.1.4 con lo0
  peer 172.16.1.4 reflect-client

  peer 172.16.1.5 as 100
  peer 172.16.1.5 con lo0
  peer 172.16.1.5 reflect-client

  peer 172.16.1.6 as 100
  peer 172.16.1.6 con lo0
  peer 172.16.1.6 reflect-client
```

PE1、PE2、P1、ASBR1、ASBR2 配置

```sql
bgp 100
  peer 172.16.1.3 as 100
  peer 172.16.1.3 con lo0
```



RR2 配置

```sql
bgp 200
  peer 172.16.1.7 as 200
  peer 172.16.1.7 con lo0
  peer 172.16.1.7 reflect-client

  peer 172.16.1.8 as 200
  peer 172.16.1.8 con lo0
  peer 172.16.1.8 reflect-client

  peer 172.16.1.10 as 200
  peer 172.16.1.10 con lo0
  peer 172.16.1.10 reflect-client

  peer 172.16.1.11 as 200
  peer 172.16.1.11 con lo0
  peer 172.16.1.11 reflect-client

  peer 172.16.1.2 as 200
  peer 172.16.1.2 con lo0
  peer 172.16.1.2 reflect-client
```

PE3、PE4、P2、ASBR3、ASBR4 配置

```sql
bgp 200
  peer 172.16.1.9 as 200
  peer 172.16.1.9 con lo0
```

RR 配置

```sql
disp bgp peer
```



##### 2. 注意： ASBR 将路由传递给 RR 的时候必须配置下一跳本地

ASBR1、ASBR2 配置

```sql
bgp 100
	peer 172.16.1.3 next-hop-local
```

ASBR3、ASBR4 配置

```sql
bgp 200
	peer 172.16.1.9 next-hop-local
```



##### 3. 配置 EBGP

ASBR1

```sql
bgp 100
	peer 10.1.57.2 as-number 200
```

ASBR3

```sql
bgp 200
	peer 10.1.57.1 as-number 100
```

ASBR2

```sql
bgp 100
	peer 10.1.68.2 as-number 200
```

ASBR4

```sql
bgp 200
	peer 10.1.68.1 as-number 100
```

查看：ASBR

```sql
disp bgp peer
```

#### 2. 配置 8、9

8. 如图3，AS100，AS200内各网元配置MPLS LSR-ID， 全局使能MPLS , MPLS LDP（已配）AS100,AS200内各有直连链路建立LDP邻居（除PE1-RR1之间，其余已配）（1）
   **上述需求与LAB-1完全一致，无视题本需求顺序，正常敲就完事了**
9. ASBR1-ASBR3、ASBR2-ASBR4之间通过直连接口建立EBGP邻居关系。在ASBR上将ISIS的loopback0口引入BGP。
   假设loopback0地址为172.16.1.Y/32，当Y为奇数时，对端设备访问本AS设备的loop back0优选的链路为ASBR1-ASBR3，
   当Y为偶数时，对端设备访问本AS设备的loopback0优选的链路为ASBR2-ASBR4，保证配置具有最好的扩展性。（10分）

分析：

1. OPTION-C2 ASBR 之间通过 `BGP-LU` 借助路由策略进行单播路由标签分配及通告
2. AS 内部继续使用 `LDP` 进行标签分配及通告，但前提要求是 ASBR 得开启特定能力 `trigger`--- 使得 LDP 可以为 BGP 标签路由继续分配标签通告给 RR
3. RR 也需要存在 IGP 路由，通过 LDP 继续分配标签，此时需要 ASBR 执行 BGP 引入进 IGP 动作

ASBR1 角度 --- trigger，bgp 引入 isis

分析：

1. 解决选路需求 --- 通过 `med`

> ​			A. ASBR 只发布环回口路由

ASBR1、ASBR3、ASBR2、ASBR4

```sql
ip ip-prefix 172 index 10 permit 172.16.0.0 16 greater-equal 32 less-equal 32
route-policy I2B permit node 10
	if-match ip-prefix 172
```

> ​		B. 要求对端 AS 访问本端 AS 设备 奇数优选 ASBR1、3，偶数优选 ASBR2、4



![image-20211008204745078](https://i.loli.net/2021/10/08/cnl6L7VjOtZW5As.png)

ASBR1 配置

```sql
acl 2000
	# 匹配奇数路由
	rule 5 permit source 172.16.1.1 0.0.0.254 
route-policy I2B permit node 5
	if-match acl 2000 
	# 将 med 设置为 0
	apply cost 0
route-policy I2B permit node 10
	if-match ip-prefix 172
# 引入路由
bgp 100
	import-route isis 1 route-policy I2B
```

ASBR2 配置

```sql
acl 2000
	# 匹配偶数路由
	rule 5 permit source 172.16.1.0 0.0.0.254 
route-policy I2B permit node 5
	if-match acl 2000 
	# 将 med 设置为 0
	apply cost 0
route-policy I2B permit node 10
	if-match ip-prefix 172
# 引入路由
bgp 100
	import-route isis 1 route-policy I2B
```



ASBR3 配置

```sql
acl 2000
	# 匹配奇数路由
	rule 5 permit source 172.16.1.1 0.0.0.254 
route-policy I2B permit node 5
	if-match acl 2000 
	# 将 med 设置为 0
	apply cost 0
route-policy I2B permit node 10
	if-match ip-prefix 172
# 引入路由
bgp 200
	import-route isis 1 route-policy I2B
```

ASBR4 配置

```sql
acl 2000
	# 匹配偶数路由
	rule 5 permit source 172.16.1.0 0.0.0.254 
route-policy I2B permit node 5
	if-match acl 2000 
	# 将 med 设置为 0
	apply cost 0
route-policy I2B permit node 10
	if-match ip-prefix 172
# 引入路由
bgp 200
	import-route isis 1 route-policy I2B
```



> C. 具有最好的扩展性

由于方案二需要将BGP路由引入进IS-IS防止路由回灌 --- 造成路由震荡问题

![image-20211008214255185](https://i.loli.net/2021/10/08/OPGZpUlhXzTenqV.png)

左边

ASBR2 将 EBGP 引入 ISIS

```sql
isis
	import-route bgp tag 200
```

ASBR1 配置路由策略 --- 加上 node1，共 3 个节点

```sql
route-policy I2B deny node 1
	if-match tag 200
```

**此时 ASBR2 - ASBR1 方向 ok 了**

ASBR1

```sql
isis
	import-route bgp tag 200
```

ASBR2

```sql
route-policy I2B deny node 1
	if-match tag 200
```

验证：查看 策略成功就 ok 了

```sql
[ASBR2]disp route-policy I2B 
Route-policy : I2B
  deny : 1 (matched counts: 13)
    Match clauses : 
      if-match tag 200
  permit : 5 (matched counts: 17)
    Match clauses : 
      if-match acl 2000
    Apply clauses : 
      apply cost 0 
  permit : 10 (matched counts: 38)
    Match clauses : 
      if-match ip-prefix 172
```

**另一个方向也 ok 了**



右边

ASBR4 将 EBGP 引入 ISIS

```sql
isis
	import-route bgp tag 100
```

ASBR3 配置路由策略 --- 加上 node1，共 3 个节点

```sql
route-policy I2B deny node 1
	if-match tag 100
```

**此时 ASBR3 - ASBR4方向 ok 了**

ASBR3

```sql
isis
	import-route bgp tag 100
```

ASBR4

```sql
route-policy I2B deny node 1
	if-match tag 100
```

**另一个方向也 ok 了**



![image-20211008215333416](https://i.loli.net/2021/10/08/RyaVLSW1rw6UmJn.png)

查看

> 如果你发现现象不对，请检查 

1. ASBR route-policy I2B
2. 下一跳本地配置
3. RR 和 P 是否进行 路由渗透

RR1 、P1配置 

```sql
ip ip-prefix 172 permit 172.16.0.0 16 greater-equal 32 less-equal 32 
isis 1 
	import-route isis level-2 into level-1 filter-policy ip-prefix 172
```

ASBR1

```sql
disp ip routing
```

![image-20211009160856648](https://i.loli.net/2021/10/09/znZyfIcJ9EuUoQP.png)

ASBR2

![image-20211009160930488](https://i.loli.net/2021/10/09/LaJndsoTv6wSjCP.png)

ASBR3

![image-20211009161029919](https://i.loli.net/2021/10/09/ucAekUdH4iw9KJy.png)

ASBR4

![image-20211009161119181](https://i.loli.net/2021/10/09/nkoRvruOJQepZBg.png)



#### OPTION C方案二

10. 如图4，各站点，通过MPLS BGP VPN跨域 **`OPTION C方案二`**，能够相互学习路由，MPLS域不能出现次优路径。（15）

##### A. ASBR 之间开启 MPLS 能力

```sql
int g0/0/2
	mpls
```

##### B. ASBR 之间进行 BGP-LU 标签通告

以 `ASBR1` 为例

1. 写一个 route-policy 分配标签

```sql
route-policy ASBR permit node 10
	apply mpls-label
```

2. 通告给邻居时候调用路由策略

```sql
bgp 100
	peer 10.1.57.2 route-policy ASBR export
```

3. 开启 `label-route-capability` 能力

```sql
bgp 100
	peer 10.1.57.2 label-route-capability
```

##### C. ASBR 配置 LDP 继续为 BGP 分配标签

ASBR 配置

```sql
mpls
	lsp-trigger bgp-label-route
```

D. 检查

在 PE1、PE2 上 查看

```
[PE1]disp mpls lsp
-------------------------------------------------------------------------------
                 LSP Information: LDP LSP
-------------------------------------------------------------------------------
FEC                In/Out Label  In/Out IF                      Vrf Name       
172.16.1.3/32      NULL/3        -/Ip-Trunk1                                   
172.16.1.3/32      1033/3        -/Ip-Trunk1                                   
172.16.1.5/32      NULL/1027     -/Ip-Trunk1                                   
172.16.1.5/32      1034/1027     -/Ip-Trunk1                                   
172.16.1.4/32      NULL/1024     -/GE0/0/0                                     
172.16.1.4/32      1035/1024     -/GE0/0/0                                     
172.16.1.6/32      NULL/1025     -/GE0/0/0                                     
172.16.1.6/32      1036/1025     -/GE0/0/0                                     
172.16.1.20/32     NULL/3        -/GE0/0/0                                     
172.16.1.20/32     1037/3        -/GE0/0/0                                     
172.16.1.1/32      3/NULL        -/-                                           
172.16.1.7/32      NULL/1029     -/Ip-Trunk1                                   
172.16.1.7/32      1046/1029     -/Ip-Trunk1                                   
172.16.1.9/32      NULL/1030     -/Ip-Trunk1                                   
172.16.1.9/32      1047/1030     -/Ip-Trunk1                                   
172.16.1.11/32     NULL/1031     -/Ip-Trunk1                                   
172.16.1.11/32     1048/1031     -/Ip-Trunk1                                   
172.16.1.10/32     NULL/1049     -/GE0/0/0                                     
172.16.1.10/32     1049/1049     -/GE0/0/0                                     
172.16.1.8/32      NULL/1050     -/GE0/0/0                                     
172.16.1.8/32      1050/1050     -/GE0/0/0                                     
172.16.1.2/32      NULL/1051     -/GE0/0/0                                     
172.16.1.2/32      1051/1051     -/GE0/0/0   
```

PE2

```sql
<PE2>disp mpls lsp
-------------------------------------------------------------------------------
                 LSP Information: LDP LSP
-------------------------------------------------------------------------------
FEC                In/Out Label  In/Out IF                      Vrf Name       
172.16.1.4/32      NULL/3        -/GE0/0/2                                     
172.16.1.4/32      1024/3        -/GE0/0/2                                     
172.16.1.20/32     3/NULL        -/-                                           
172.16.1.6/32      NULL/1025     -/GE0/0/2                                     
172.16.1.6/32      1025/1025     -/GE0/0/2                                     
172.16.1.3/32      NULL/1033     -/GE0/0/0                                     
172.16.1.3/32      1026/1033     -/GE0/0/0                                     
172.16.1.5/32      NULL/1034     -/GE0/0/0                                     
172.16.1.5/32      1027/1034     -/GE0/0/0                                     
172.16.1.1/32      NULL/3        -/GE0/0/0                                     
172.16.1.1/32      1028/3        -/GE0/0/0                                     
172.16.1.7/32      NULL/1046     -/GE0/0/0                                     
172.16.1.7/32      1046/1046     -/GE0/0/0                                     
172.16.1.9/32      NULL/1047     -/GE0/0/0                                     
172.16.1.9/32      1047/1047     -/GE0/0/0                                     
172.16.1.11/32     NULL/1048     -/GE0/0/0                                     
172.16.1.11/32     1048/1048     -/GE0/0/0                                     
172.16.1.10/32     NULL/1032     -/GE0/0/2                                     
172.16.1.10/32     1049/1032     -/GE0/0/2                                     
172.16.1.8/32      NULL/1033     -/GE0/0/2                                     
172.16.1.8/32      1050/1033     -/GE0/0/2                                     
172.16.1.2/32      NULL/1034     -/GE0/0/2                                     
172.16.1.2/32      1051/1034     -/GE0/0/2 
```

PE3 --- 可以发现都是走的是 `g0/0/2`，那是因为 P2 --- RR2 的 ospf cost = 10，而 PE3 --- PE4 ospf cost = 20

PE4 也是如此

![image-20211009224645580](https://i.loli.net/2021/10/09/GNknyzoVhTKAj7m.png)

```sql
<PE3>disp mpls lsp
-------------------------------------------------------------------------------
                 LSP Information: LDP LSP
-------------------------------------------------------------------------------
FEC                In/Out Label  In/Out IF                      Vrf Name       
172.16.1.2/32      NULL/3        -/GE0/0/0                                     
172.16.1.2/32      1032/3        -/GE0/0/0                                     
172.16.1.11/32     3/NULL        -/-                                           
172.16.1.7/32      NULL/1024     -/GE0/0/2                                     
172.16.1.7/32      1025/1024     -/GE0/0/2                                     
172.16.1.9/32      NULL/3        -/GE0/0/2                                     
172.16.1.9/32      1026/3        -/GE0/0/2                                     
172.16.1.8/32      NULL/1025     -/GE0/0/2                                     
172.16.1.8/32      1027/1025     -/GE0/0/2                                     
172.16.1.10/32     NULL/1029     -/GE0/0/2                                     
172.16.1.10/32     1028/1029     -/GE0/0/2                                     
172.16.1.5/32      NULL/1032     -/GE0/0/2                                     
172.16.1.5/32      1033/1032     -/GE0/0/2                                     
172.16.1.1/32      NULL/1033     -/GE0/0/2                                     
172.16.1.1/32      1034/1033     -/GE0/0/2                                     
172.16.1.3/32      NULL/1034     -/GE0/0/2                                     
172.16.1.3/32      1035/1034     -/GE0/0/2                                     
172.16.1.20/32     NULL/1035     -/GE0/0/2                                     
172.16.1.20/32     1036/1035     -/GE0/0/2                                     
172.16.1.6/32      NULL/1036     -/GE0/0/2                                     
172.16.1.6/32      1037/1036     -/GE0/0/2                                     
172.16.1.4/32      NULL/1037     -/GE0/0/2                                     
172.16.1.4/32      1038/1037     -/GE0/0/2   
```

##### D. PE 将 vpnv4 路由传递给 RR

注意：**RR 关闭 RT 值检测**

> 1. HUB 端：RR1 - PE1 - PE2 建立 vpnv4 邻居
>

RR1 配置

```sql
bgp 100
	ipv4-family vpnv4
		undo policy vpn-target
    peer 172.16.1.1 enable
    peer 172.16.1.1 reflect-client
    peer 172.16.1.20 enable
    peer 172.16.1.20 reflect-client
```

PE1、PE2 配置

```sql
bgp 100
	ipv4-family vpnv4
  	peer 172.16.1.3 enable
```

检查：

RR1 上 disp bgp vpnv4 all peer

要看到 `.1 .20`

> 2. SPOKE 端：RR2  - PE3 - PE4  建立 vpnv4 邻居 
>

RR2 配置

```sql
bgp 200
ipv4-family vpnv4
	undo policy vpn-target
  peer 172.16.1.2 enable
  peer 172.16.1.2 reflect-client
  peer 172.16.1.11 enable
  peer 172.16.1.11 reflect-client
```

这里为什么 RR2 为啥要指定 PE3、4 为客户端？

spoke 端确实没有互访要求，那就不需要配置

![image-20211013235805575](https://i.loli.net/2021/10/13/x5mN74Rztu1FQ9G.png)

增值业务 - 复用 - 扩展性 

但如果 PE 又增加了其它 CE 场景，且 CE 需要互访怎么办，此时就需要借助 `reflect-client` 了

![image-20211013235901988](https://i.loli.net/2021/10/13/cfpnhdzHGyoK3J7.png)

PE3、PE4 配置

```sql
bgp 200
ipv4-family vpn-instance VPN1 
	import-route ospf 10
ipv4-family vpnv4
    peer 172.16.1.9 enable
```

此时就应该能学到本端的 vpnv4 路由了

```sql
<RR2>disp bgp vpnv4 all peer 
  Peer            V          AS  MsgRcvd  MsgSent  OutQ  Up/Down       State PreRcv
  172.16.1.2      4         200       58      118     0 00:54:02 Established    3
  172.16.1.3      4         100       88       62     0 00:54:49 Established    34
  172.16.1.11     4         200       23       42     0 00:18:39 Established    3
```

##### E. RR 之间建立 MP-EBGP 多跳传递 vpnv4 路由

注意：RR2 配置 `allow-as-loop`

RR 之间**关闭**传递 `ipv4 unicast`路由

RR1

```sql
bgp 100 
 ipv4-family vpnv4
		peer 172.16.1.9 enable
  	peer 172.16.1.9 next-hop-invariable
 ipv4-family unicast
  	undo peer 172.16.1.9 enable
```

RR2

```sql
bgp 200
  ipv4-family vpnv4
		peer 172.16.1.3 enable
    peer 172.16.1.3 next-hop-invariable 
    peer 172.16.1.3 allow-as-loop
  ipv4-family unicast
    undo peer 172.16.1.3 enable
```

检查：disp bgp vpnv4 all peer

此时 RR 上可以查看到对端 vpnv4 路由了 -长 as-path

```sql
<RR2>disp bgp vpnv4 all routing-table
Route Distinguisher: 22:22
*>   172.17.1.1/32      172.16.1.20                           0      100 65000?
*>   172.17.1.2/32      172.16.1.20                           0      100 65000?
*>   172.17.1.3/32      172.16.1.20                           0      100 65000 100 200?
*>   172.17.1.4/32      172.16.1.20                           0      100 65000 100 200?
```



##### F. 避免次优问题，要求配置下一跳不变

RR1

```sql
bgp 100
  ipv4-family vpnv4
    peer 172.16.1.1 next-hop-invariable
    peer 172.16.1.9 next-hop-invariable
    peer 172.16.1.20 next-hop-invariable
```

RR2

```sql
bgp 200
  ipv4-family vpnv4
    peer 172.16.1.11 next-hop-invariable
    peer 172.16.1.3 next-hop-invariable
    peer 172.16.1.2 next-hop-invariable
```

PE3 上查看下一跳

```sql
<PE3>disp bgp vpnv4 all rou
*>i  172.17.1.1/32      172.16.1.1                 100        0      100 65000?
* i                     172.16.1.20                100        0      100 65000?
*>i  172.17.1.2/32      172.16.1.1                 100        0      100 65000?
* i                     172.16.1.20                100        0      100 65000?
*>   172.17.1.3/32      0.0.0.0         2                     0      ?
* i                     172.16.1.1                 100        0      100 65000 100 200?
* i                     172.16.1.20                100        0      100 65000 100 200?
*>i  172.17.1.4/32      172.16.1.1                 100        0      100 65000 100 200?
* i                     172.16.1.20                100        0      100 65000 100 200?
<PE3>disp ip rou vpn-instance VPN1 
	172.17.1.1/32  IBGP    255  0          RD   172.16.1.1      GigabitEthernet0/0/2
  172.17.1.2/32  IBGP    255  0          RD   172.16.1.1      GigabitEthernet0/0/2
  172.17.1.3/32  OSPF    10   1           D   10.3.33.2       Mp-group0/0/0
  172.17.1.4/32  IBGP    255  0          RD   172.16.1.1      GigabitEthernet0/0/2
```

PE4

```sql
<PE4>disp ip rou vpn-instance VPN1 
	172.17.1.1/32  IBGP    255  0          RD   172.16.1.20     GigabitEthernet0/0/2
  172.17.1.2/32  IBGP    255  0          RD   172.16.1.20     GigabitEthernet0/0/2
  172.17.1.3/32  IBGP    255  0          RD   172.16.1.20     GigabitEthernet0/0/2
  172.17.1.4/32  OSPF    10   1           D   10.3.34.2       GigabitEthernet0/0/1
```

CE3 - CE4 可以互通了

```sql
<CE3>tracert -a 172.17.1.3 172.17.1.4
 traceroute to  172.17.1.4(172.17.1.4), max hops: 30 ,packet length: 40,press CT
RL_C to break 
 1 10.3.33.1 40 ms  10 ms  20 ms 
 2 10.1.119.1 160 ms  90 ms  110 ms 
 3 10.1.79.1 100 ms  110 ms  100 ms 
 4 10.1.57.1 90 ms  80 ms  110 ms 
 5 10.1.35.1 100 ms  120 ms  100 ms 
 6 10.2.11.5 110 ms  130 ms  80 ms 
 7 10.2.11.6 110 ms  110 ms  90 ms 
 8 10.2.11.1 110 ms  120 ms  100 ms 
 9 10.1.12.2 250 ms  210 ms  210 ms 
10 10.1.24.2 230 ms  240 ms  210 ms 
11 10.1.46.2 210 ms  230 ms  180 ms 
12 10.1.68.2 230 ms  250 ms  250 ms 
13 10.1.81.2 240 ms  210 ms  230 ms 
14 10.3.34.1 220 ms  230 ms  230 ms 
15 10.3.34.2 230 ms  220 ms  230 ms
```



11. CE1-PE1之间链路开，CE1设备仍可以学习到spoke业务网段。配置保障有最好的扩展性。（6）

12. 在拓扑正常情况下，要求CE1，CE2访问spoke网段时，不从本AS内绕行。（1）

13. 在PE3，PE4上修改BGP local-preference属性，实现CE3,CE4访问非直接的10.3.x.0/24网段时，

    若X为奇数，PE3，PE4优选的下一跳为PE1；

    若X为偶数，PE3，PE4优选的下一跳为PE2，不用考虑来回路径是否一致。（3分）

**上述与 LAB1 一致！**



#### Future（11分）

4.1 HA（8分）

1. CE1配置静态的默认路由访问ISP，下一跳为100.0.1.2.，该默认路由的NQA ICMP测试绑定，每隔5s测试执行一次（2）

CE1 配置 

##### NQA 

```sql
nqa test-instance admin icmp 
 	test-type icmp
 	destination-address ipv4 100.0.1.2
 	# 频率 修改为 15 为了看到现象
	frequency 15
 	start now
```

 默认路由联动

```sql
ip route-static 0.0.0.0 0.0.0.0 100.0.1.2 track nqa admin icmp
```

查看结果

`disp nqa results` --- 看到 `Completion:success`

但是考试的配置频率为 `5`，结果为 `Completion:no result  `

```sql
  5 . Test 24 result   The test is finished
   Send operation times: 3              Receive response times: 3          
   Completion:success                   RTD OverThresholds number: 0       
   Attempts number:1                    Drop operation number:0            
   Disconnect operation number:0        Operation timeout number:0         
   System busy operation number:0       Connection fail number:0           
   Operation sequence errors number:0   RTT Status errors number:0         
   Destination ip address:100.0.1.2                                      
   Min/Max/Average Completion Time: 20/20/20                             
   Sum/Square-Sum  Completion Time: 60/1200                              
   Last Good Probe Time: 2021-10-14 11:32:56.0                           
   Lost packet ratio: 0 %  
```

1. CE2，CE3,CE4能够通过默认路由访问ISP（4）

4.2 NAT（2分）

 1. 在CE1上，10.3.0.0/16（不含10.3.2.10）的内网地址转换为102.0.1.2-102.0.1.6，通过Ge2/0/0访问ISP。sever1拥有单独的公网地址102.0.1.1，对ISP提供FTP和HTTP（2）

4.3 QOS（7分）

 1. 在CE1和G2/0/0的出方向，周一至周五的8：00-18：00点对TCP目的端口号6881-6999流量，承诺平均速率为1Mbps（3）

**上述与 LAB1 一致！**

#### 5 IPV6组播（14分）

![image-20211007161442136](https://i.loli.net/2021/10/14/oprBI4idcXFUCZk.png)

1. ISIS-IPv6和如图配置IPv6接口开销值（5分）
2. AS 100中相邻设备建立PIM IPV6 SM的邻居关系。PE1的E0/0/0静态加入组FF1E::AA。（2分）

注意：以上 6 台设备一定要打开 isis 的多拓扑能力

```sql
sy
isis
	ipv6 enable topology ipv6
	quit
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
disp ipv6 routing pro isis # 看到 D1 to D6 可以验证下 cost 是否为 2510
```



> 1. 以 PE1 为例

建立邻居关系

```sql
multicast ipv6 routing-enable
int g0/0/0
	pim ipv6 sm 
int ip-trunk1
	pim ipv6 sm
```

查看邻居关系

`disp pim ipv6 neighbor`

![image-20211014115404678](https://i.loli.net/2021/10/14/lvmHNKQr4hdZgDI.png)

> 2. 加组

PE1的接口静态加组，需要报文PE1的接口IPv6可以工作（接口为up/up）, PE1 连接一台 pc

```sql
interface e0/0/0
	ipv6 enable
	ipv6 address auto link-local
 	mld static-group FF1E::AA
```

`disp ipv6 int brief`

3. ASBR1和ASBR2的loopback0为C-BSR且都为FF1E::/112的C-RP。ASBR1的loopback0口成为BSR，ASBR1为RP确保PIM IPV6 SM域生成（*,G）表项无次优路径。（3分）

A、选举 BSR，通过多台 C-BSR 选举唯一 BSR，优先级比大、优先级一致 C-BSR 地址比大

ASBR1 配置

1. 指定 loop0 

```sql
pim-ipv6
 c-bsr 2000:EAD8:99EF:C03E:B2AD:9EFF:32DD:DC05
```

2. loop0 接口开启 `pim ipv6 sm`

```sql
int loop0
	pim ipv6 sm
```

3. 调整 `c-bsr` 优先级

```sql
pim-ipv6
	 c-bsr priority 255
```

ASBR2 配置

1. 指定 loop0 

```sql
pim-ipv6
 c-bsr 2000:EAD8:99EF:C03E:B2AD:9EFF:32DD:DC06
```

2. loop0 接口开启 `pim ipv6 sm`

```sql
int loop0
	pim ipv6 sm
```

检查

`disp pim ipv6 bsr-info` --- 可以看到 `Elected BSR Address: DC05`

```sql
<PE1>disp pim ipv6 bsr-info 
 VPN-Instance: public net
 Elected AdminScoped BSR Count: 0
 Elected BSR Address: 2000:EAD8:99EF:C03E:B2AD:9EFF:32DD:DC05
     Priority: 255
     Hash mask length: 126
     State: Accept Preferred
     Scope: Not scoped
     Uptime: 00:04:18
     Expires: 00:01:52
     C-RP Count: 0
```



**B、配置C-RP的服务组地址范围，并且调整ASBR1为RP --- 越小越优**

ASBR1 配置

1. acl 匹配出服务组范围

```sql
acl ipv6 2000
	rule 5 permit source FF1E:: 112
```

2. C-RP 调用 组策略

```sql
pim-ipv6
	c-rp 2000:EAD8:99EF:C03E:B2AD:9EFF:32DD:DC05 group-policy 2000
```

3. 修改 C-RP 优先级为 0

```sql
pim-ipv6
	c-rp priority 0
```

ASBR2 配置

1. acl 匹配出服务组范围

```sql
acl ipv6 2000
	rule 5 permit source FF1E:: 112
```

2. C-RP 调用

```sql
pim-ipv6
	c-rp 2000:EAD8:99EF:C03E:B2AD:9EFF:32DD:DC06 group-policy 2000
```

查看

ASBR 之间接口抓包

![image-20211014184906060](https://i.loli.net/2021/10/14/wiI7bU9Xuz6vcDe.png)

```sql
<PE1>disp pim ipv6 rp-info ff1e::aa
 VPN-Instance: public net
 BSR RP Address is: 2000:EAD8:99EF:C03E:B2AD:9EFF:32DD:DC05
     Priority: 0
     Uptime: 00:06:13
     Expires: 00:02:10
 RP mapping for this group is: 2000:EAD8:99EF:C03E:B2AD:9EFF:32DD:DC05
```



确保PIM IPV6 SM域生成（*,G）表项无次优路径----RPT没有次优路由 

假设没有部署路由泄露，组播RPT出现次优路径，PE1会RP建立RPT最优上游为P1（没有明细路由）

![image-20211014185841266](https://i.loli.net/2021/10/14/V8epgJMbskcN5xd.png)

RR1 和 P1 配置

1. ipv6-prefix 匹配出 ipv6 地址

```sql
ip ipv6-prefix LOOP index 10 permit 2000:EAD8:99EF:C03E:B2AD:9EFF:32DD:DC00 120 greater-equal 128 less-equal 128
```

2. 进行路由渗透

```sql
isis 1
	ipv6 import-route isis level-2 into level-1 filter-policy ipv6-prefix LOOP
```

检查 看到 PE1 的上游接口为 `Upstream interface: Ip-Trunk1` 就 ok 了

```sql
<PE1>disp pim ipv6 routing-table 
 VPN-Instance: public net
 Total 1 (*, G) entry; 0 (S, G) entry

 (*, FF1E::AA)
     RP: 2000:EAD8:99EF:C03E:B2AD:9EFF:32DD:DC05
     Protocol: pim-sm, Flag: WC EXT 
     UpTime: 00:31:16
     Upstream interface: Ip-Trunk1
         Upstream neighbor: FE80::7D6E:0:CA21:1
         RPF prime neighbor: FE80::7D6E:0:CA21:1
     Downstream interface(s) information: None
```

