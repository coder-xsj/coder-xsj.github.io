
> 情况 1：只有一段开启 mtu 检查

| 设备   | rotuer-id | mtu-enable                | mtu  | mtu 协商结果 | ospf 状态机 |
| ------ | --------- | ------------------------- | ---- | ------------ | ----------- |
| 第一组 |           |                           |      |              |             |
| R1     | 1.1.1.1   | 开启                      | 1500 | 通过         | Full        |
| R2     | 2.2.2.2   | 不开启（发送的 mtu 为 0） | 1400 | 通过         | Full        |
| 第二组 |           |                           |      |              |             |
| R1     | 1.1.1.1   | 不开启                    | 1400 | 通过         | Full        |
| R2     | 2.2.2.2   | 开启                      | 1500 | 通过         | Full        |

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

分析原因：由于 MASTER 的 MTU 小，所以 MASTER 不能通过 MTU 检查，直接卡在 EXSTART 阶段。SLAVE 通过了 MTU 检查，并且开始发送有内容的 DD 报文，所以卡在 EXCHANGE 阶段。

分析原因：由于 SLAVE 的 MTU 小，所以 MASTER 可以通过 MTU 检查，等 SLAVE 发送有内容的  DD。`但 SLAVE 不能通过 MTU 检查，所以不会主动送有内容的 DD`，这样两者都卡在 EXSTART 阶段。





