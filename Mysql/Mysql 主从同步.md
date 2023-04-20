### 主备复制
- Mysql 通过 Binlog 进行主备复制
- Mysql 主备库之间维持了一个长连接，主库中有一个线程专门服务备库B的长连接
	- 备库通过 change master 命令，设置主库A 的 ip、端口、用户名、密码，以及那个位置开始请求binlog（偏移量）
	- 备库执行 start slave 命令，启动两个线程：
		- io_thread：负责与主库建立连接
		- sql_thread
	- 备库拿到 binlog 后，写到本地文件，称为中转日志（relay log）
	- sql_thread 读取中转日志，解析日志命令，并执行

主备流程图
![](https://mynoteimage.oss-cn-beijing.aliyuncs.com/note/2022-04-05-145005.jpg)


### Binlog 三种格式
- statement
	- 记录 sql 语句原文
	- 可能会产生主备数据不一致
		- delete from t where a > 4 and t_modified = '2018-11-10' limit 1
		- 主备库选择的索引可能不一样
- row
	- 记录了真实删除行的主键id
		- table_map event：记录操作的表
		- Delete_rows event：定于删除行为
- mixed
	- statement 格式会可能导致主备不一致，所以用 row 格式
	- row 格式很占空间，一条语句操作多行的话，会有很多条记录
	- mixed 格式，Mysql 会判断该语句是否可能引起主备不一致
		- 有可能则， row 格式
		- 否则，statement 格式

### 循环复制问题
- 实际生产更多的是双 M架构，即互为主从备份
- 循环复制问题
	- 节点 A 执行语句，节点 B 执行完后，生成 binlog
	- 节点 A 同时是节点 B 的备库，就会循环复制执行
- 如何解决循环复制
	- Binlog 会记录命令第一次执行的实例 server id 
	- 规定两个库的 server id 必须不同，如果相同，不能为主备关系
	- 备库接到 binlog 重放的过程中，生成与原 binlog 的server id 相同的新的 binlog 
	- 每个库收到主库发来的日志后，先判断 server id，如果跟自己相同，则直接丢弃