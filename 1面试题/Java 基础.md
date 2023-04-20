### 集合类是怎么解决高并发中的问题
- 哪些是非安全
- 普通的安全集合类
- JUC

### 自定义异常
### Object 定义的方法
- toString 
- hashCode
- equals
- clone
- finalized
	- 
- wait
- clone
	- 返回对象的副本
		- 深克隆
		- 浅克隆

### 1.8 新特性
- Lambada 表达式
- 函数式接口
- 方法引用、构造器调用
- Stream API
- 接口中的默认方法和静态方法
- 新时间日期 API

### Stream 并行操作原理
fork/join 
- 创建 Stream
- 中间操作
- 终止操作

### fork/join 

### 动态代理

- CGlib： 面向父类的动态代理

### equals == 区别
- == 运算符
- equals 只能引用类型

- hashCode 
	- 保证同一个类的不同对象不一样
	- 重写 equals 必须重写 hashCode 方法：equals 相同时，应保证hashCode 是一致的
- equals ture 
- 