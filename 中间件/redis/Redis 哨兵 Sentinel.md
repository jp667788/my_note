### Redis 哨兵高可用
主从复制模式下，注解点出现故障后，需要转移主节点：
- 主节点故障后，连接主节点失败
- 主节点连接失败，需要选出一个新的 slave 节点，执行 slaveof no one，成为新的主节点
- salve 节点变成主节点后，更新应用放主节点信息
- 客户端命令另一个节点 slave2 复制新的主节点
- 待原来的主节点恢复后，复制新的主节点


当主节点出现故障时，哨兵能自动完成故障发现和故障转移：
- 主节点故障后，连接主节点失败
- 每个 Sentinel 节点定期监控发现主节点故障
- 多个 Sentinel 节点对主节点故障达成一致，选举出新的主节点
- Sentinel 领导者节点执行故障转移

### 哨兵实现原理
#### 定时监控任务
1. 每隔 10 秒，每个 sentinel 节点向主节点发送 info 命令 获取最新的拓扑结构
	- 通过主节点获取 info 信息，不需要给 Sentinel 显示配置从节点信息
	- 有新的节点加入可以立刻感知
	- 节点不可达或者故障转移，info 命令可以实时更新

2. 每隔 2 秒，每个 Sentinel 节点会向 Redis 数据节点的 _sentinel_ ：hello 频道，发送**该 Sentinel 节点对于主节点的判断**和 **当前 Sentinel 节点的信息**。同时，Sentinel 节点也会订阅该频道：

	- 发现新的 Sentinel 节点：Sentinel 节点订阅后，可以了解其他 Sentinel 节点的信息，如果是先加入的节点，保存起来
	- Sentinel 节点之间交换主节点的状态，作为客观下线和 leader 选举的依据

3. 每个 1 秒，每个 Sentinel 节点会向主节点、从节点、其余 Sentinel 节点发送 ping 命令，进行心跳检查，来确认这些节点是否可到达。
	
#### 主观下线 & 客观下线
- 主观下线
	- Sentinel 发心跳检查，没有有效回复时，该 Sentinel 节点就会对该节点做失败判定
-  客观下线
	-  如果是**主节点主观下线**，该 Sentinel 节点会通过 sentinel is-master-down-by-addr 命令向其他 Sentinel 节点询问对主节点的判断。当数量超过  \<quorum\> 后，则被判为客观下线


#### Sentinel Leader 选举
Redis 使用 Raft 算法实现领导选举
1. 每个在线的 Sentinel 节点都有资格称为 leader，确认主观下线后，向其他节点发送 sentinel is-master-down-by-addr 命令，要求自己设置为领导者
2. 收到命令的 Sentinel 节点，如果没同意其他节点，则同意请求，否则拒绝
3. 如果该 Sentinel 节点发现自己票数已经大于等于 Max(quorum， num(sentinel/2 + 1))，则称为 leader
4. 如果此过程没有选出领导者，则进行下一次

> 如果哨兵集群中只有2个实例，哨兵称为 Leader 必须获取 2 票。如果此时有一个哨兵挂了，那么就无法完成主从切换。
> 在判断主观下线后，就进入了选举流程。只有进入到选举流程的哨兵，才能自己给自己先投一票，如果它还没有进入选举流程，只能给别人投票

#### 故障转移
1. 在从节点列表中选出一个节点作为新的主节点，方法如下：
	1. 过滤：不健康（主观下线、断线）、5秒内没回复过 Sentinel 节点 ping 响应、与主节点断连超过 down-after-millseconds * 10秒
	2. 选择 slave-priority （从节点优先级）最高的从节点列表，存在返回，不存在继续
	3. 选择复制偏移量最大的节点（复制最完整）、
	4. 选择 rundId 最小的节点
	<img src="https://mynoteimage.oss-cn-beijing.aliyuncs.com/note/2022-03-16-140626.png" width="50%">
2. Sentinel Leader 对选出来的从节点，执行 slaveof no one 命令，使其成为主节点
3. Sentinel Leader 节点向剩余从节点发送命令，使其成为新主节点的从节点
4. Sentinel 节点集合奖原来的主节点更新从节点，其回复后，命令其去复制新的节点
