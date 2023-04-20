## redo log (重做日志)
redo log 是引擎层日志

WAL 技术 Write-Ahead Logging ：写日志，再写磁盘。

当有一条记录需要更新时，InnoDB 引擎会先把记录写到 redo log 中，并更新内存。在适当的时候，再根据日志，把记录更新的磁盘中去。

InnoDB 中的 redo log 是 **固定大小** ，从头开始写 写到末尾就回到开头循环写。

![](https://tva1.sinaimg.cn/large/008i3skNgy1gt8ata3ls2j30vq0nsaaq.jpg)

### write pos 
write pos 是当前记录的位置，往后移动。

### check point 
check point 是当前擦除的位置，也是往后推移并循环擦除记录前 要把记录更新到数据文件中。


write pos 后和  check pos 前的区域是空白区域，可以记录新的数据。
redo log 保证了即使 InnoDB 异常，之前未写入数据文件中的记录不会丢失（crash-safe）。


## bin log
bin log 是 Server 层日志

bin log 是 归档日志， 没有 crash-safe 的功能

## bin log 和 redo log 的区别
1. redo log 是引擎层；bin log 是 Server 层，可以多个引擎共用。
2. redo log 是物理日志，记录了”某个数据页做了什么修改。bin log 是逻辑日志，记录了 sql 的具体逻辑，比如：把 id =2 的 c 字段 +1 
3. redo log 是循环写，空间固定，写完会擦除。bin log 可以追加写入，达到一定大小会往下一个文件写，不会覆盖


## 执行一条更新语句的流程
```
UPDATE T SET c = c + 1 WHERE id = 2;
```

1. 执行器向引擎获取 id=2 的记录，如果不在内存，需要在磁盘中写入内存。
2. 执行器把新的值 c+1 请求引擎更新。
3. 引擎更新新的值到内存中后，写入 prepare 状态的 redo log ，返回执行器更新完成，随时可以提交事务。
4. 执行器写入 bin log
5. 执行器调用引擎提交事务接口，引擎将 prepare 状态的 redo log 改为 commit 状态，更新完成。

![](https://tva1.sinaimg.cn/large/008i3skNgy1gt8b9xgirvj30u013zq4v.jpg)


## 两阶段提交

redo log 第一次写入为 prepare 状态，当 bin log 写入成功后，更改为 commit 状态，这就是两阶段提交。

两阶段提交是保证 redo log 和 bin log 的一致性。

如果没有两阶段提交：
1. **先写 redo log ，再写 bin log** ：在写完 redo log 后，bin log 还没开始写的时候，Mysql 异常重启。重启后，由于 redo log 已经有这条操作的记录，那这个时候 Mysql 会回复刚刚的操作，但是此时 bin log 没有记录。如果靠 bin log 恢复数据时，就会缺少这一条记录。
2. **先写  log ，再写 redo log** ：在写完 bin log 后，redo log 还没开始写的时候，Mysql 异常重启。重启后，由于 redo log 没有这条操作的记录，Mysql 不会恢复刚刚的操作，但是此时 bin log 有记录。


当有了两阶段提交：
	1. redo log 2. bin log 3. commit
	
1. 2 之前崩溃，redo log 没有 commit，也没有对应的 bin log，回滚该事务。
2. 3 之前崩溃，redo log 没有 commit，但是有对应的 bin log ，自动提交该事务。

## 脏页 flush

### flush
内存里的数据写入磁盘的过程，称为 flush。
在 flush 之前，内存文件和磁盘数据页内容会不一致。

### 脏页
内存文件和磁盘数据页内容不一致的情况，称为脏页。
内存文件和磁盘数据页一致，称为干净页。

脏页、干净页都在内存中


### flush 的时机
1. redo log 满了。系统会停止所有更新操作，将 [[#check point]] 往前推进 redo log 留出空间。

	![](https://mynoteimage.oss-cn-beijing.aliyuncs.com/note/2021-10-27-140208.png)
	
	check point 从 cp 到 cp' 时，会将此区间（绿色部分）内所有的脏页都刷新到磁盘中。
	
2. 系统内存不足。此时需要淘汰一些数据页，如果淘汰的是 脏页 ，就要现将脏页刷新到磁盘中。
> 为什么不是直接删除内存，从磁盘读取数据页时，再从 redo log 中恢复数据？
> - 可以保证在内存里的一定是正确的结果，可以直接返回。
> - 内存里没有的数据，数据文件一定是正确的结果，加载到内存后就可以直接返回。

3. 系统空闲时间
4. Mysql 正常关闭

### flush 脏页对性能的影响
1. redo log 满了
	整个系统不能再接受更新，更新操作会被堵塞。
	写性能为0.


2. 内存不够用。
	InnoDB 使用 缓冲池管理内存 （Buffer Pool ），当读入的数据页没有在内存中，需要向缓冲池中申请内存后，内存不够的情况下。只能把**最久不使用**的内存淘汰掉
	
	内存页有三种状态：
	- 未使用过的，直接可以使用
	- 使用过的干净页，释放后直接使用
	- 使用过的脏页，必须先 flush 到磁盘
要淘汰的脏页个数太多，会影响性能

### InnoDB flush 脏页的策略
- innodb_io_capacity 参数。

	越小代表系统能力越差，刷脏页就会越慢
	
- innodb_max_dirty_pages_pct 参数，脏页比例上限。默认 75%
	
	InnoDB 会根据当前脏页比例（设为M），计算出 0 - 100 的数字（设为 F1(M) 函数）
	
	F1（M）:
	``` java
		if (M>=innodb_max_dirty_pages_pct then) {
			return 100;
		}
      	return 100*M/innodb_max_dirty_pages_pct;
	```
	
	InnoDB 每次写入 redo log 都会有一个序号，当前序号和 checkpoint 之间的差值，可以理解为当前写入 redo log 的位置和 checkpoint 之间的距离，设为N。计算的函数 设为 F2(N)。
	
	**然后取 F1(M) 和 F2(N) 计算结果中的较大值，记为 R，之后 InnoDB 就会按照 innodb_io_capacity * R% 来控制 flush 速度。**	

> InnoDB 在刷脏页时，如果旁边的数据页也是脏页，就会一起刷掉，而且会传递下去。可以通过 innodb_flush_neighbors 来控制：1 打开，0 关闭。
> 在机械硬盘时代，这种机制会有效减少随机IO，提升系统性能。
> SSD 或者 IOPS 比较高，这个就不需要打开。


### Binlog 写入机制
事务执行过程中：
- 先把日志写入 Binlog cache 
- 再把 Binlog cache 写入binlog 文件
- 清空 Binlog cache 


#### binlog cache
- 每个线程会有一个 binlog cache ，共用同一个 binlog file
- binlog_cache_size 控制大小
- 如果超过 binlog_cache_size 需要暂存到磁盘

![](https://mynoteimage.oss-cn-beijing.aliyuncs.com/note/2022-04-05-100803.jpg)

- write：日志写入文件系统的 page cache
- fsync：数据持久化到磁盘的动作，一般情况下，我们认为 fsync 才占磁盘的 IOPS。

#### write 和  fsync 的时机
由 sync_binlog 控制：
- sync_binlog = 0：每次提交事务，只 write 不 fsync
- sync_binlog = 1：每次提交事务，都会执行 fsync
- sync_binlog =N(N>1)：每次事务提交，只write，累积到 N 个事务后才 fsync
	- 风险：主机异常，可能丢失N个数据的

### redo log 写入日志
- 先写入 redo log buffer
- 再持久化到 redo log 磁盘
- 事务提交时，会持久化到磁盘
- 有可能会被提前持久化，也就是事务没提交的时候， redo log buffer 已经被持久化到磁盘


#### redo log 的 三种状态
![](https://mynoteimage.oss-cn-beijing.aliyuncs.com/note/2022-04-05-101809.png)

- redo log buffer：mysql 进程中的内存
- 写到磁盘（write），但是没有持久化到磁盘（fsync）: 文件系统的 page cache
- hard disk: 持久化到磁盘

#### redo log 写入控制
innodb_flush_log_at_trx_commit 参数控制 redo log 的写入策略
- 设为 0：每次事务提交都只是把 redo log 留在 redo log buffer 中
- 设为 1 :  每次事务提交都把  redo log 持久化到磁盘
- 设为 2：每次事务提交都只是把 redo log 写到 page cache

InnoDB 后台线程，每隔 1 s，会把 redo log buffer 中的 日志，调用 write 写入文件系统的 page cache，然后调用 fsync  持久化到磁盘

> 注意：事务执行过程中产生的 redo log 是直接写在 redo log buffer 中的，也可能会被后台线程持久化，出现还没有提交的事务的 redo log 被持久化到磁盘中。
> 除了这种情况，还会有两种情况导致未提交的事务的 redo log 写入：
> - redo log buffer 的空间即将达到 innodb_log_buffer_size 一半的时候，后台线程会主动写盘
> - 并行事务提交时，顺带持久化了还未提交事务的 redo log buffer

#### 组提交






