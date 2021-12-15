#### 跨域C1-ASBR上没有分配对应标签

我的步骤如下：

已经重做了一遍，然后还对照的老秦题解的配置

我把贴出来，求帮忙找下问题

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
到这一步，现象是一样的

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

![image-20211002211536122](https://i.loli.net/2021/10/06/IQq1EoujwR6VeGr.png)

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



但此时 RR 上查看 `bgp-label` 还是没有，怎么回事呢？发现 ASBR 之间的接口**没有开启 `mpls` 功能**

![image-20211002211255441](https://i.loli.net/2021/10/06/MSdbvDfoTHWUOzt.png)

ASBR1-4

```
int g0/0/2
	mpls
```

此时 ASBR、RR 上再查看

以 ASBR1、RR1 为例

已经可以看到 为对端路由 重新分配标签了

**就这一步不一样的，没有奇数 ASBR1 分配、偶数 ASBR2 分配**

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

这是老秦的现象正确的

![image-20211006131525581](https://i.loli.net/2021/10/06/Vma7W9gxy1fvj24.png)

另外：

 `ASBR` 指向  `RR` 配置 `next-hop-local`也配置了

有哪位大佬有时间帮忙看下不？





解决方案是，在 RR1 --- ASBR1 配置 

```sql
int g0/0/1
	isis cost 1000
```

![image-20211006143441346](https://i.loli.net/2021/10/06/aCpeLdS4zPlTH3h.png)

此时 ASBR1 分配标签正确

```sql
[ASBR1]disp bgp routing-table label
				Network           NextHop           In/Out Label 
 													#	走的是 ASBR3
 *>     172.16.1.7        10.1.57.2         1035/1036
 													#	走的是 ASBR2
 *>i    172.16.1.8        172.16.1.6        1040/1038
 *                        10.1.57.2         NULL/1031

 *>     172.16.1.9        10.1.57.2         1039/1032
 *>i    172.16.1.10       172.16.1.6        1038/1040
 *                        10.1.57.2         NULL/1033
 *>     172.16.1.11       10.1.57.2         1037/1034
 *>     172.16.1.20       10.1.35.1         1031/NULL
```



 
