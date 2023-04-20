

## 概述
Synchronized 关键字是 Java 内建的一种同步机制，是一种 Java 内置的原子锁。当一个线程对共享资源加锁后，在其释放锁之前，其他想要访问该共享资源的线程都需要等待。

synchronized 具有 独占、互斥、排他的特性。

## Synchronized 的使用
- 修饰实例方法
	synchronized 修饰实例方法时，线程获取的是实例对象中的锁（[[#Monitor]]）。

- 修饰代码块
	synchronized 持有指定对象的锁
	
- 修饰静态方法
	synchronized 配合 static 关键字使用， 线程持有该类 .class 的锁
	
## 原理分析
### 加锁和释放锁


### 加锁对象 
- Object 对象
- 不能用 String 常量 Integer Long 
	- 

## Monitor 对象
> 任何对象都关联了一个管程，管程是控制对象并发访问的一种机制。可以理解 synchronized 关键字是 Java 中对管程的实现
	
	