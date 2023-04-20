### 主从库模式
Redis 提供了主从库模式，以保证数据副本的一致，主从库之间采用的是读写分离的方式。
- 读操作：主库、从库都可以接收；
- 写操作：首先到主库执行，然后，主库将写操作同步给从库。

### 主从复制
#### 复制功能实现
Redis 2.8 开始 使用 **PSYNC** 命令执行复制同步

- 完整重同步 full resynchronization
	- 用于处理初次复制情况
		- 服务器向主服务器发送 PSYNC \<runid\> \<offset\>命令
		- 主服务器执行 BGSAVE，创建 RDB 文件
		- 使用一个缓冲区 （repl buffer）记录现在所有的写命令
		- BGSAVE 执行完成后，主服务器发送 RDB 文件
		- 从服务器接受后，清空数据库，并载入RDB 文件
		- 主服务器记录在缓冲区里面的所有命令发送从服务器

<img src= "https://mynoteimage.oss-cn-beijing.aliyuncs.com/note/2022-03-14-152820.jpg" width="80%">

- 部分重同步 partial resynchronization
	- 用于断线后重复制

- PSYNC 部分重同步模式解决了旧版复制功能断线后低效情况

#### 部分重同步实现
- 主服务器的复制偏移量（replication offset）
- 主服务器的复制积压缓存区（replication buffer）
- 服务器运行id (run ID)

#### 复制偏移量 replication offset 
- 主从服务器分别维护一个 offset 
- 主服务器每次向服务器传播 N 个字节的数据，offset 加上 N
- 从服务器每收到 N 个字节的数据，offset 加上 N

![](https://mynoteimage.oss-cn-beijing.aliyuncs.com/note/2022-03-14-153656.png)

#### 复制积压缓冲区
主服务器维护一个固定长度（fixed-size） 先进先出（FIFO）队列，默认 1MB
- 主服务器发送命令同时，会将命令发写入 replication buffer
- 从服务器断线重连后
	- 如果从服务器 offset 之后的数据存在 replication buffer 中，执行部分重同步
	- 如果不存在，则执行完整重同步

#### 服务器运行 ID
- 每个 Redis 服务器，都有自己的 run ID
- run ID 在服务器启动时自动生成，40个随机十六进制字符
- 从服务器进行初次复制时，会保存主服务器 run ID
- 从服务器断线重连进行同步时，主服务器判断从服务器中保存的主服务器 run ID
	- 如果与当前主服务器 run ID 一致，可以尝试执行部分重同步
	- 如果不一致，执行完全重同步

### PSYNC 命令实现
- PSYNC 命令格式：PSYNC \<runid\> \<offset\>
- 从服务器请求
	- 第一次复制从服务器向主服务器发送 PSYNC ？ -1 
	- 不是第一次则发送 PSYNC \<runid\> \<offset\>，runid 是上次复制主服务器的 run ID
- 主服务器响应
	- 返回 +FULLRESYNC \<runid\> \<offset\>，标识主从间执行完全重同步
	- 返回 +CONTINUE，表示主从间执行部分重同步
	- 返回 -ERR 
