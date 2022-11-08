# VPC

>  需求：一个 VPC 里面，两个 Pulic Subnet
>
>  Public1: demo-a、demo-b
>
>  Public2: demo-c、demo-d
>
>  实现让 demo-a ping 通 demo-c 的私网 ip、demo-b ping 不通 demo-c 的私网 ip
>
>  
>
>  在一个 VPC 下的两个 subnet 中（subnet - A，subnet-B) ，subnet-A 和 subnet- B 各子 subnet 中允许互通，只允许 subnet -A 中的 A 主机 访问 subnet-B 中的 C 主机；

### VPC 规划

| ip              | stubnet | EC2            |
| --------------- | ------- | -------------- |
| 192.168.1.0/24  | Public1 | demo-a、demo-b |
| 192.168.16.0/24 | Public2 | demo-c、demo-d |

![1.png](https://s2.loli.net/2022/10/31/Cwy49hfqAQBUEVu.png)

### EC2 

| 名称   | 私网 ip        | 公网 ip        | 子网    |
| ------ | -------------- | -------------- | ------- |
| demo-a | 192.168.1.249  | 13.114.185.188 | Public1 |
| demo-b | 192.168.1.14   | 3.112.206.158  | Public1 |
| demo-c | 192.168.16.40  | 52.193.221.12  | Public2 |
| demo-d | 192.168.16.154 | 18.183.193.227 | Public2 |

做法：

## 方案一：只需要再 demo-c 的安全组中只允许 demo-a 的私网 ip 访问即可 --- 不行

![2.png](https://s2.loli.net/2022/10/31/zq1pNvGFewulSjc.png)

测试：

![3.png](https://s2.loli.net/2022/10/31/ZiTlEI6mkWGzHbf.png)

![4.png](https://s2.loli.net/2022/10/31/E1I9NhLXF2VYset.png)

**经过测试后，忽略掉一个问题，就是 demo-c 只放行 demo-a 的 icmp 的话，也会拒绝掉同子网 public-2 下的 demo-d**

![5.png](https://s2.loli.net/2022/10/31/WUDlbvK6OTioCeM.png)

------

## 方案二：通过 VPC 的网络 acl 规则实现

1. 先还原下 demo-c 的安全组，入站规则的 icmp 让其可以全部访问

![6.png](https://s2.loli.net/2022/10/31/p42vIfZRzrioN6k.png)

2. 创建一个 acl 规则 `allowDemoA` 关联 demo-vpc 下的 public-2 子网

![7.png](https://s2.loli.net/2022/10/31/WMmcKV85rPaIAZb.png)

3. 编辑入站规则

![8.png](https://s2.loli.net/2022/10/31/Ln1Mmy25DZud9ak.png)

4. 测试

测试 ssh 窗口分别为 demo-a、demo-b、demo-d

![vpc.gif](https://s2.loli.net/2022/10/31/Kvg3FCphTJMiX79.gif)