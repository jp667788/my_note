### 进程与线程的区别

- 进程： 进程是系统资源分配和调度的一个独立单位。java语言中，“正在执行程序的主体”称为线程。
- 线程： 线程是进程中的一个实体，线程共享进程的所有资源，线程只拥有很少的系统资源。一个线程中可以有多个线程。 

### 线程的状态

- 新建状态(**new**)
	- 创建线程是的状态，此时线程还没有开始运行。
- 就绪(可运行)状态(**Runnable**)
	- 执行线程，调用县城的strat()方法后，线程进入就绪状态，等待cpu的调度。
- 运行状态(**Running**)
	- 当获得cpu调度时，线程就真正的调用run()方法，进入了运行状态。
- 阻塞状态(Blocked)
	- 线程会因为各种原因进入阻塞状态：
		- 线程调用sleep方法；
		- 线程调用了一个在I/O上在阻塞的操作，即I/O操作完成之前不会返回到它的调用者；
		- 线程试图获得一个锁，但是这个锁正在被别的线程所拥有；
		- 线程进入阻塞状态并没有终止运行，暂时让出cpu。
- 死亡状态（Terminated）
	- 有两个原因会导致线程死亡：
		- run方法正常退出而自然死亡。
		- 一个未捕获的异常终止了run方法而使线程猝死。
	
![](https://mynoteimage.oss-cn-beijing.aliyuncs.com/note/2022-04-17-101405_MqmF_1859679.jpg)

#### 等待队列 
线程休息室（等待队列只是一个虚拟的概念，并不是一个存在的字段）

所有实例都拥有一个等待队列，它是在实例的wait方法执行后停止操作的线程的队列。执行wait方法之后，线程会暂停操作，进入等待队列。除非发生下列的某一种情况，否则线程会一直在等待队列中休眠。

- 有其他线程的notify方法唤醒线程。
- 有其他线程的notifyAll方法唤醒线程。
- 有其他线程的einterrupt方法唤醒线程
- wait方法超时

#### wait方法 
将线程放入等待队列
wait方法会将线程放入等待队列中，调用wait()或者this.wait()。**需要注意的是，执行wait()方法时，线程必须持有锁，如果线程进入了等待队列，线程会释放对象的锁。**

#### notify方法 
从等待队列中取出线程。

- notify方法会讲等待队列中的一个线程取出。
- 取出的线程并不会立即执行，需要重新获取被对象的锁，如果别的线程正在持有该对象的锁，须等待该线程释放了锁对象之后重新获得。
- 需注意的是。如果等待队列中有多个等待被唤醒的线程，notify()方法选择线程的顺序、规则是不固定的，这取决于Java运行平台的环境。

#### notifyAll()方法 
从等待队列中取出所有的线程

- notifyll() 方法会讲所有在等待队列的线程都唤醒，这些线程会竞争锁资源。
- notfyAll()和notify()方法和wait() 方法一样，执行该方法的线程必须拥有对象的锁，否则会抛出 java.lang.IllegaMonitorStateException。

 注意：wait() notify() notifyAll() 方法是Object类的方法，并不是Thread类中的方法，也就是说每一个对象都拥有这几个方法，所以对应的：**每个对象都拥有一把锁和一个等待队列**。

### 线程的启动
- 继承Thread类，创建实例启动线程。
	- new Thread.start()
- 实现Runnable接口，创建实例启动线程。
	- 创建Ruannable的实例，将实现类传递给Thread作为构造参数，再调用Thread的start方法。
- 实现 Callable 接口
	- 创建Callable实例，传递给  FutureTask 作为构构造参数，再调用Thread的start方法。
	
>注意：
>1. 线程实例创建之后，线程不会自动执行。线程结束之后，线程实例也不会消失。
>2. 程序中如果多个线程，主程序结束之后，并不代表着程序终止，等到程序中所有的线程都执行结束之后，程序才会终止。
> 程序终止是指处守护线程（Daemon Thread） 以外的线程全部终止。可以通过setDaemon设置为守护线程。
> 3. Thread 类本身也实现了Runnable接口，并且持有run方法，但Thread类的run方法是一个空方法。需要子类重写。
> 4. 可以用ThreadFactory创建线程并启动

### 监视器(monitor)
- java中每个对象都拥有唯一的一个监视器，在非多线程编码时该监视器不发挥作用，反之如果在synchronized 范围内，监视器发挥作用。
- 重量级锁的情况下，[[Java 对象模型#Mark Word]]会有一个指针指向 Monitor
- 拥有监视器的有下面几种办法：
	- 执行该对象的同步方法：线程获取Thread类的monitor

	``` java
	public class Thread implements Runnable {
		public sychronized void xxx(){
			....
		}
	}
	```
	
	- 执行某个对象的同步快：线程获取obj实例的monitor

	``` java
	public class Thread implements Runnable {
		Object obj;
		public void xxx() {
			sychronized(obj){
				...
			}
		}
	}
	```

	- 执行某个类的静态同步方法：获取这个类所有实例的monitor

	``` java
	public class Thread implements Runnable {
		public static sychronized xxx() {
			...
		}
	}
	```

> 注意：拥有monitor的是线程
> 1. 同时只能有一个线程可以获取某个对象的monitor
> 2. 一个线程通过调用某个对象的wait()方法释放该对象的monitor并会进入休眠状态，直到其他线程获取该对象呢的monitor，或调用该对象的notify()或notifyAll()后，再次竞争后获取该对象的monitor。
> 3. 只有拥有该对象的monitor的线程才可以调用该对象的notify()或notifyAll()方法。如果没有拥有该对象的monitor的线程调用了该对象的notify()或notifyAll()会抛出异常(java.lang.IllegalMonitorStateException)。


### 创建线程的方式
1. extends Thread
2. implements Runnable
3. implements Callable

### 启动线程
- new Thread.start()
- new Thread(new Runnable).start()

#### Thread.run() vs Thread.start()
- run()
- start()：异步

### 主要方法
- sleep()
- yeid()
	- 让出执行权
- join()
	- 自己调自己没什么用
	- t1 中 t2.join，t1 等待 t2 执行完继续运行
- 