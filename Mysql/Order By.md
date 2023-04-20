## Order By

表定义：

``` mysql
CREATE TABLE `t` ( 
	`id` int(11) NOT NULL, 
	`city` varchar(16) NOT NULL, 
	`name` varchar(16) NOT NULL, 
	`age` int(11) NOT NULL, 
	`addr` varchar(128) DEFAULT NULL, 
	PRIMARY KEY (`id`), 
	KEY `city` (`city`)
) EN
```

查询 sql :

``` mysql
select city,name,age from t where city = '杭州' order by name limit 1000 ;
```

### sort buffer
Mysql Server 层 会给每个线程分配一个内存用于排序，这个内存区域被称为 sort buffer 。当排序结束后，会释放 sort buffer

### 全字段排序
explain 后， Extra 中含有 file sort 代表采用的全字段排序。

#### 执行步骤：
1. 初始化 sort buffer，确定需要存放字段 city, name, age
2. 根据 city 索引树，查询第一个满足 city = '杭州' 的数据的 id
3. 再根据 id 去 主键索引树，查询所有字段，取出 city,name,age 三个字段放入 sort_buffer 中
4. 再继续在 city 索引树中 查询下一个
5. 重复3，4 直到 city='杭州' 不满足
6. 在 sort_buffer 中 根据 name 做快速排序
7. 按照排序结果取出前 1000 个 返回客户端

![](https://mynoteimage.oss-cn-beijing.aliyuncs.com/note/2021-11-13-075655.jpg)

sort_buffer 的内存大小由 sort_buffer_size 控制。如果数据大小小于 sort_buffer_size，排序操作就在内存中完成。如果数据大于 sort_buffer_size 就需要借助磁盘临时文件辅助排序。

数据会被分到多个磁盘的临时文件上，采用**归并排序**方法，先分别对每个文件上的数据进行排序，再合并到一起并再次排序。

### rowid 排序
explain 后， Extra 中没有含有 file sort。（但是 rowid 也会用到 临时文件）

当单行返回的字段很多的时候，也需要生成大量的磁盘临时文件进行辅助排序，性能会下降。

这种情况 mysql 会进行rowid 排序，即sort_buffer 中只存储主键Id和需要排序的字段（name），降低临时文件的数量。之后执行器再去主键索引表中，获取整行数据返回。

#### 执行步骤：
1. 初始化 sort_buffer ，此时只需要存放 id 和 name 字段
2. 根据 city 索引树，查询第一个满足 city = '杭州' 的数据的 id
3. 再根据 id 去 主键索引树，查询所有字段，取出 ,name,id 两个字段放入 sort_buffer 中
4. 再继续在 city 索引树中 查询下一个
5. 重复3，4 直到 city='杭州' 不满足
6. 在 sort_buffer 中 根据 name 做快速排序
7. 再根据 id 去主键索引树查询整行数据，取出 city,name,age 三个字段返回客户端

![](https://mynoteimage.oss-cn-beijing.aliyuncs.com/note/2021-11-13-081107.jpg)

> 结果集只是一个逻辑概念，Mysql 不会再花费内存来存储结果，而是查询到一条就给客户端返回一条


### 全字段排序 VS rowid 排序

- 如果内存足够大，会优先选在全字段排序
- 内存不够大时，才会选在 rowid 排序
- rowid 需要回表操作，会造成较多的磁盘读，优先级较低
- 如果需要排序的字段上有索引且被使用，则不需要进行排序

### 使用索引字段
建立联合索引 （city，name）

#### 执行步骤
1. 根据 city,name 索引树，查询第一个满足 city = '杭州' 的数据的 id
2. 再根据 id 去 主键索引树，查询所有字段，取出 city，name 作为结果集的一部分直接返回
3. 再继续在 city,name 索引树中 查询下一个
4. 重复 2，3 ，直到查询到 1000个数据或者 city='杭州' 不满足

### 使用索引字段（覆盖索引）
建立联合索引 （city，name，age）
#### 执行步骤
1. 根据 city,name ,age 索引树，查询第一个满足 city = '杭州' 的数据的 id，取出 city,name,age 作为结果集的一部分直接返回
2. 再继续在 city,name,age 索引树中 查询下一个
3. 重复 2，3 ，直到查询到 1000个数据或者 city='杭州' 不满足


### 临时表
``` mysql
	CREATE TABLE `words` ( 
		`id` int(11) NOT NULL AUTO_INCREMENT, 
		`word` varchar(64) DEFAULT NULL, PRIMARY KEY (`id`)
	) ENGINE=InnoDB;
	
	-- 插入10000行数据
	-- ...
	
	select word from words order by rand() limit 3;
```

![](https://mynoteimage.oss-cn-beijing.aliyuncs.com/note/2021-11-14-062521.jpg)

explain 结果中， Extra 字段中包含 **Using temporary**， 代表使用了**临时表**。
Using filesort 代表需要排序。
Extra 代表 使用了临时表，并需要对临时表排序


> 由于调用了 rand() 函数，Mysql 需要给每行数据调用 rand() 函数后，根据 rand() 的结果进行排序，所以需要存储每行数据和 rand() 之后的值。

内存临时表中，由于在内存中，回表不用访问磁盘，所以临时表的排序优先使用 [[#rowid 排序]]


#### 执行步骤
``` mysql
	select word from words order by rand() limit 3;
```

1. 创建临时表，使用 Memory 引擎，表中记录 rand() 的结果(R)  和 word (W) 字段两列。（这个表没有索引）
2. 从 Words 表中按主键顺序取出所有 word 值，调用 rand() 函数，分别记录临时表 W，R 字段，扫描行数 10000
3. 临时表中存在 10000 行数据，对此表进行排序
4. 初始化 sort_buffer，采用 row_id 排序，sort_buffer 记录 rand()值 和 主键（位置信息）
5. 从内存临时表中取出每行数据，取出 rand() 值和位置信息放入 sort_buffer ，全表扫表内存表，扫描行数 +10000 。
6. sort_buffer 根据 rand() 值排序
7. 取出前三条位置信息，去内存表中查找整行数据，取出 word 值返回客户端。扫描行数 + 3

![](https://mynoteimage.oss-cn-beijing.aliyuncs.com/note/2021-11-14-064940.jpg)

> row_id 其实代表每个引擎用来唯一标识数据行的信息。
> - 有主键的 InnoDB 表，row_id 就是主键 ID
> - 无主键的 InnoDB 表，row_id 系统自动生成
> - Memory 不是索引组织表，类似数组，row_id 就是数组下标

#### 磁盘临时表

tmp_table_size 默认 16M，超过则从内存临时表转成磁盘临时表。
磁盘临时表默认引擎是 InnoDB，由 internal_tmp_disk_storage_engine 参数控制 

### 优先队列排序
临时文件的算法属于**归并排序算法**

tmp_table_size 不够时，sort_buffer 充足（ n 条数据），且包含 limit n 字句，此时Mysql 会**优先使用优先队列排序，而不是使用磁盘临时表**。 

#### 执行步骤
1. 先取前三行数据，构造一个堆
2. 取下一行 R'，跟当前堆中最大的 R 比较，如果 R' 小于 R，去除 R 替换成 R'。
3. 重复 2 ，知道 10000 行数据遍历结束

![](https://mynoteimage.oss-cn-beijing.aliyuncs.com/note/2021-11-14-071819.jpg)


