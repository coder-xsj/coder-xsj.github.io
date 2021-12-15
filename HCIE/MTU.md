> 情况 1：只有一段开启 mtu 检查
>

| 设备   | mtu-enable | mtu  | mtu 协商结果 | ospf 状态机 |
| ------ | ---------- | ---- | ------------ | ----------- |
| 第一组 |            |      |              |             |
| R1     | 开启       | 1500 | 通过         | Full        |
| R2     | 不开启     | 1400 | 通过         | Full        |
| 第二组 |            |      |              |             |
| R1     | 不开启     | 1400 | 通过         | Full        |
| R2     | 开启       | 1500 | 通过         | Full        |

> 情况 2：两端同时开启 mtu 检查
>
> MASTER.MTU < SLAVE.MTU
>
> MASTER.MTU > SLAVE.MTU

| 设备   | MASTER/SALVE | mtu  | mtu 协商结果 | ospf 状态机 |
| ------ | ------------ | ---- | ------------ | ----------- |
| 第一组 |              |      |              |             |
| R1     | MASTER       | 1400 | 不通过       | ExStart     |
| R2     | SLAVE        | 1500 | 通过         | ExChange    |
| 第二组 |              |      |              |             |
| R1     | MASTER       | 1500 | 通过         | ExStart     |
| R2     | SLAVE        | 1400 | 不通过       | ExStart     |

分析原因：由于MASTER的MTU小，所以MASTER不能通过MTU检查，直接卡在EXSTART阶段。SLAVE通了MTU检查，并且开始发送有内容的DBD报文，所以卡在EXCHANGE阶段。

分析原因：由于SLAVE的MTU小，所以MASTER可以通过MTU检查，等SLAVE送有内容的DBD。`但SLAVE不能通过MTU检查，所以不会主动送有内容的DBD`，这样两者都卡在EXSTART阶段。





