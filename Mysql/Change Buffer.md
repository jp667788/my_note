## 前提 
在一个表中插入一条数据，这张表有主键索引和普通索引。在插入这条数据时，同时需要往这两个索引树中插入索引数据。

对于主键索引：一般主键索引都是自增的，不需要磁盘随机读取。（UUID或者指定的值，不一定是自增，还是需要随机读取）

对于普通索引：普通索引不是顺序插入了，需要随机读取磁盘的索引数据，导致了性能下降。

为了提升性能，对于**非唯一索引**，在每次执行更新语句时，
- 如果当前需要修改的记录的数据页在内存中
	- 直接更新内存
- 如果不在内存中
	- 放入 Chang Buffer 中
	- Mysql 返回结果
	- 在适当实际讲 Change Buffer 写入磁盘中

**Change Buffer 也会进行持久化**

## Change Buffer 
### Change Buffer 的使用条件
Change Buffer 只有在下面条件时才会使用：
- 索引是辅助索引（ 不是主键索引）
- 索引是非唯一索引
唯一索引 Mysql 需要判断插入的数据是否是唯一的，此时必须要将磁盘数据读入内存进行判断。

### Merge Change Buffer 
将 Change Buffer 应用到 数据页的过程称为: Merge Change Buffer

在下面几种情况会进行 Merge Change Buffer :
- 该数据页被加载到缓冲池（访问该数据页）
- 当索引页没有可用空间时（空余空间<1/32页）
- Mysql 系统线程定时 merge 
- Mysql 正常关闭 （shutdown）


### Change Buffer 和 redo log

记入 Change Buffer 的操作也会被 记入 redo log 中

执行下面语句，Mysql 中流程如下图

``` mysql
	insert into t(id,k) values(id1,k1),(id2,k2);
```
![](https://mynoteimage.oss-cn-beijing.aliyuncs.com/note/2021-10-23-072146.jpg)

### 扩展
- Change Buffer 的结构是 B+ 树
- 存放在系统表空间中 （ibdata1）
- 非叶子节点存放 search key ，表示记录所在表的表空间 id
