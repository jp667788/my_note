### 过期删除策略
- 当 Redis 内存超出物理内存限制，内存数据会和磁盘开始产生频繁的交换（Swap），交换会让 Redis 的性能急剧下降
- 为了限制最大使用内存，Redis 提供了配置参数 maxmemory 来限制内存超出期望大小
- 当内存超出 maxmemory Redis 提供了几种可选策略

- noeviction
	- 不会继续写请求，DEL 请求可以继续。
	- 保证数据不丢失，但是线上任务不能持续进行
	- 默认淘汰策略
- volatile-lru
	- 尝试淘汰设置了过期时间的 key
	- 最少使用的 key 优先淘汰
	- 没有设置过期时间的 key 不会淘汰
- volatile-ttl
	- 尝试淘汰设置了过期时间的 key
	- ttl 剩余值越小优先被淘汰
- volatile-random
	- 尝试淘汰设置了过期时间的 key
	- 过期 key 集合中随机的 key删除
- allkeys-lru
	- 淘汰全体的 key 集合
	- 最少使用的 key 优先淘汰
- allkeys-random
	- 淘汰全体的 key 集合
	- 全体 key 集合中随机的 key删除


### 近似 LRU 算法
- 实现 LRU 算法除了需要 key/value 字典外，还需要附加一个链表
	- 链表中的元素按照一定的顺序排列
	- 会消耗大量的额外内存
- Redis 使用一种近似 LRU 算法，
	- 每个 key 增加了一个额外的小字段，长度 24 bit
	- LRU 淘汰只有懒惰处理
		- 写操作时发现内存超出 maxmemory，执行 LRU
		- 随机采样出 n (可以配置) 个 key，
		- 淘汰掉旧的 key
		- 淘汰后依然超出，继续随机采样淘汰，直到低于 maxmemeory
		- 
	