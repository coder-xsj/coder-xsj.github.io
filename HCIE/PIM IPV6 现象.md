#### PIM IPV6 现象

拓扑

![image-20211014195012349](https://i.loli.net/2021/10/14/kBT4hzdW7m2enoa.png)

我是这么做的

ASBR2 配置

```sql
interface GigabitEthernet0/0/3
	ipv6 enable
	ipv6 address 2000:EAD8:99EF:C03E:B2AD:9EFF:32DD:DCC0/127
	isis ipv6 enable 1
	pim ipv6 sm
```

PIM-test 配置

1. 配置接口地址

```sql
interface GigabitEthernet0/0/3
	ipv6 enable
	ipv6 address 2000:EAD8:99EF:C03E:B2AD:9EFF:32DD:DCC1/127
	isis ipv6 enable 1
	pim ipv6 sm
	
interface LoopBack0
 	ipv6 enable
 	ipv6 address 2001::DCC0/128
 	isis ipv6 enable 1
	 pim ipv6 sm
```

2. 配置 isis

```sql
isis 1
 cost-style wide
 network-entity 49.0002.0000.0000.0013.00
 ipv6 enable topology ipv6
```

3. 检查 邻居

A、isis

```sql
[PIM-test]disp isis peer 

                          Peer information for ISIS(1)

  System Id     Interface          Circuit Id       State HoldTime Type     PRI
-------------------------------------------------------------------------------
0000.0000.0006* GE0/0/3            0000.0000.0006.03 Up   8s       L2       64 

Total Peer(s): 1
```

B、pim ipv6 

```sql
[PIM-test]disp pim ipv6 neighbor 
 VPN-Instance: public net
 Total Number of Neighbors = 1

 Neighbor        Interface           Uptime   Expires  Dr-Priority  BFD-Session
 FE80::5689:98FF:FE6D:3EB1
                 GE0/0/3             00:25:04 00:01:42 1            N  
```

4. ping 组播源

```sql
[PIM-test]ping ipv6 FF1E::AA -i LoopBack 0
  PING FF1E::AA : 56  data bytes, press CTRL_C to break
    Request time out
    Request time out
    Request time out
    Request time out
    Request time out

  --- FF1E::AA ping statistics ---
    5 packet(s) transmitted
    0 packet(s) received
    100.00% packet loss
    round-trip min/avg/max = 0/0/0 ms

[PIM-test]ping ipv6 FF1E::AA -i GigabitEthernet 0/0/3
  PING FF1E::AA : 56  data bytes, press CTRL_C to break
    Request time out
    Request time out
    Request time out
    Request time out
    Request time out

  --- FF1E::AA ping statistics ---
    5 packet(s) transmitted
    0 packet(s) received
    100.00% packet loss
    round-trip min/avg/max = 0/0/0 ms
```



5. PE1 路由 --- 发现有 PIM-test 的 loop0 接口地址

![image-20211014194044797](https://i.loli.net/2021/10/14/AOkmVgIy73zlbHN.png)

6. PIM-test 设备 的 bsr-info，rp-info

```sql
[PIM-test]disp pim ipv6 bsr-info 
 VPN-Instance: public net
 Elected AdminScoped BSR Count: 0
 Elected BSR Address: 2000:EAD8:99EF:C03E:B2AD:9EFF:32DD:DC05
     Priority: 255
     Hash mask length: 126
     State: Accept Preferred
     Scope: Not scoped
     Uptime: 00:34:24
     Expires: 00:01:46
     C-RP Count: 2
[PIM-test]disp pim ipv6 rp-info 
 VPN-Instance: public net
 PIM-SM BSR RP Number:2
 Group/MaskLen: FF1E::/112
     RP: 2000:EAD8:99EF:C03E:B2AD:9EFF:32DD:DC05
     Priority: 0
     Uptime: 00:34:32
     Expires: 00:01:58
 Group/MaskLen: FF1E::/112
     RP: 2000:EAD8:99EF:C03E:B2AD:9EFF:32DD:DC06
     Priority: 192
     Uptime: 00:34:32
     Expires: 00:01:58
```

求怎样能 ping 通，想看下 disp pim ipv6 routing

