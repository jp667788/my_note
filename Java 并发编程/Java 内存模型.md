# Java 内存模型 Java Memory Model


## 概述
Java虚拟机规范视图定义一种**Java内存模型（Java Memory Model,JMM**）来屏蔽各种硬件和操作系统的内存访问的差异，以实现让java程序在各种不同的平台都能达到一致的内存访问结果。

它**描述了Java程序中各种变量(线程共享变量)的访问规则**，以及在JVM中将变量存储到内存中和从内存中读取出变量这样的底层细节。

本身是一种抽象的概念，实际上并不存在，它描述的是一组规则或规范，通过这组规范定义了程序中各个变量（包括实例字段，静态字段和构成数组对象的元素）的访问方式。

## 关键词
- **可见性**：一个线程对共享变量值的修改，能及时被其他线程看到。
- **共享变量**：如果一个变量在多个线程的工作内存中存在副本，那么这个变量就是这几个线程的共享变量。

## 主内存与工作内存
Java内存模型规定了所有的变量都存储在主内存（Main Memory）。

每个线程还有自己的工作内存，线程的工作内存中保存了被该线程使用的变量在主内存中的副本，线程对所有变量的操作都必须在工作内存中进行，而不能直接操作主内存中的变量；

不同线程之间无法直接访问其他线程的工作内存中的变量，线程之间变量值的传递需要通过主内存来完成。

![线程、主内存、工作内存之间的交互关系](https://mynoteimage.oss-cn-beijing.aliyuncs.com/note/2021-09-04-57DC62E8-37C6-4659-941D-D7FB1BA2EDA2.png)
线程、主内存、工作内存之间的交互关系

## 共享变量的可见性
如果要实现共享变量可见，必须保证以下两点：
-   线程修改后的共享变量值能够及时从工作内存中刷新到主内存中。
-   其他线程能够及时的把共享变量最新的值从主内存中更新到自己的工作内存中。
    

### 共享变量不可见原因
   - 线程的交叉执行
   - 重排序结合线程交叉执行
   - 共享变量的值更新后没有及时的在工作内存和主内存中刷新

### synchronized
synchronized 是 Java在**语言层面实现可见性**

JMM规定了使用了synchronized的线程：
1. 线程解锁前，必须把共享变量的最新值刷新到主内存中。
2. 线程加锁时， 将清空工作内存中的共享变量的值，从而使用共享变量时，需要从主内存中重新读取最新的值。

以上两点保证了线程解锁前对共享变量的修改在下次加锁时对其他线程可见。

![线程执行互斥代码的顺序](https://mynoteimage.oss-cn-beijing.aliyuncs.com/note/2021-09-04-39FCEB31-28D4-4BFD-829C-12E68922DCA2.png)
	
	
## volatile
把对volatile变量 的单个读/写，看成是使用同一个监视器锁对这些单个读/写操作做了同步。

### 特性
- 可见性：对一个volatile变量的读，总是能看到（任意线程）对这个volatile变量最后的写入。
- 原子性：对任意单个volatile变量的读/写具有原子性，但是volatile++这种操作并不具有原子性。
        
### 如何解决volatile++操作的原子性问题

- 使用synchronized关键字
- 使用ReentrantLock(java.until.concurrent.locks)
- 使用AtomicInteger(vava.util.concurrent.atomic)

### volatile写-读建立的happens before关系 

线程A写volatile变量，线程B读volatile变量。线程A写volatile之前的所有可见共享变量，在线程B读volatile变量之后对线程B全部可见。
        
### volatile写/读的内存语义
    
- 当写一个volatile变量时，JMM会把该线程对应的工作内存中的共享变量刷新到主内存中。
- 当读一个volatile变量时，JMM会把该线程的工作内存置为无效，重新从主内存中读取共享变量。
- 线程A写一个volatile变量，随后线程B读这个volatile变量，这个过程实质上是线程A通过主内存向线程B发送消息。
        
### volatile语义的实现
 为了实现volatile的内存语义，JMM 限制了对 volatile 变量操作的重排序(编译器重排序、处理器重排序)，并且会在指令序列中插入内存屏障来禁止特定类型的处理器重排序，我们看先先后有两个操作

#### 限制指令重排序
- 当第二个操作为volatile写时，第一个操作不论是什么都不允许重排序。
- 当第一个操作为volatile读时，第二个操作不论是什么都不允许重排序。
- 当第一个操作为volatile读，第二个操作为volatile写时，不允许重排序。

#### 内存屏障（处理器）
    
- 在每个volatile写操作的前面插入一个StoreStore屏障。（禁止前面的普通写和下面的volatile写操作重排序)
- 在每个volatile写操作的后面插入一个StoreLoad屏障。（禁止前面的volatile写操作与下面可能有的volatile读/写操作重排序）
- 在每个volatile读操作的后面插入一个LoadLoad屏障。（禁止后面所有的普通读操作与volatile读操作重排序）
- 在每个volatile读操作的后面插入一个LoadStore屏障。（禁止后面所有的普通写操作与volatile读操作重排序）

![](https://mynoteimage.oss-cn-beijing.aliyuncs.com/note/2021-09-12-055333.jpg)
    
![](https://mynoteimage.oss-cn-beijing.aliyuncs.com/note/2021-09-12-062009.jpg)


处理器会根据不同情况对内存屏障进行优化，省略部分屏障。
![](https://mynoteimage.oss-cn-beijing.aliyuncs.com/note/2021-09-12-061843.jpg)

  > 注意：最后的 StoreLoad 屏障不能省略。因为第二个 volatile 写之后，方法立即 return。此时编译器可能无法准确断定后面是否会有 volatile 读或写，为了安全起见，编译器常常会在这里插入一个 StoreLoad 屏障。

### volatile变量的适用场景
    
 - 对变量的写入操作不依赖其当前值。
	 - 不满足：volatile++，volatile=volatile * 2;
	 - 满足：boolean类型 
- 该变量没有包含在具有其他变量的不变式中。
	- 不满足：不变式 volatile1 < volatile2
            
### volatile 和synchronized比较
- volatile不需要加锁，比synchronized更轻量级，不会阻塞线程。
- 从内存可见性的角度，volatile读相当于加锁，volatile写相当于解锁。
- synchronized可以保证原子性和可见性，但是volatile对于类似于volatile++的操作无法保证可见性。
  
## final
final 域的读和写更像是普通变量访问
    
### final 重排序规则
        
- 在构造函数内，对一个final域的写入，与随后把这个被构造的对象的引用赋值给一个引用变量，这两个顺序不能重排序。
- 初次读一个包含final域的对象的引用，与随后初次读这个final域，这两个操作之间不能重排序。
            
	#### 写final域的重排序规则
	- JMM禁止编译器把 final 域的写重排序到构造函数之外（普通变量会）            
	- 编译器会在 final 写之后，在构造函数return之前，插入一个Storestrore的屏障，禁止处理器final域的写重排序。
	>  写 final 与的重排序规则会保证，在对象引用为任一线程可见之前，final 与已经被正确的初始化了，而普通域不具有这个保障。
            
	#### 读final域的重排序规则

	- 在一个线程中，初次读对象引用与初次读取该对象包含的final域，JMM禁止处理器重排序这两个操作。（仅仅针对与处理器），编译器会在final域操作的前面插入Loadload内存屏障。

	#### final 语义在处理器中的实现
	- 写 final 域规则要求：编译器在 final 域的写止呕，构造函数 return 之前，插入 StoreStore 屏障。
	- 读 final 域规则要求：编译器在读 final 于的操作前面插入一个 LoadLoad 内存屏障

### MESI

 ## 内存屏障指令
| 屏障类型 | 指令示例 | 说明 |
| - | - | - |
| LoadLoad | Load1; LoadLoad; Load2 | 确保 Load1 数据的装载，之前于Load2 及所有后续装载指令的装载。 |
| StoreStore | Store1; StoreStore;Store2 |确保 Store1 数据对其他处理器可见（刷新到内存），之前于Store2 及所有后续存储指令的存储。 |
| LoadStore | Load1; LoadStore;Store2 | 确保 Load1 数据装载，之前于Store2 及所有后续的存储指令刷新到内存。 |
| StoreLoad | Store1; StoreLoad;Load2 | 确保 Store1 数据对其他处理器变得可见（指刷新到内存），之前于Load2 及所有后续装载指令的装载。StoreLoad Barriers 会使该屏障之前的所有内存访问指令（存储和装载指令）完成之后，才执行该屏障之后的内存访问指令。 |

> StoreLoad Barriers 是一个“全能型”的屏障，它同时具有其他三个屏障的效果。

## 重排序

 为了使处理器内部的 运算单元能尽量充分利用，处理器可能会对输入代码进行乱序执行优化。

### 数据依赖性   

 如果两个操作访问同一个变量。且这两个操作中有一个为写操作，此时这两个操作之间就存在数据依赖性。数据依赖分为三种类型
-   写后读；a = 1;b = a; 写一个变量之后，再读这个位置。
-   写后写；a = 1;a = 2; 写一个变量之后，再写这个变量。s
-   读后写；a = b;b = 1; 读一个变量之后，再写这个变量。
  

 **编译器和处理器在重排序时，会遵守数据依赖性，编译器和处理器不会改变存在数据依赖关系的两个操作的执行顺序**。

  

### as-if-serial语义
    
 不管怎么重排序，（单线程）程序的执行结果不能被改变。编译器、runtime、处理器都必须准星as-id-serial语义。 但在多线程程序中，对存在控制依赖的操作重排序，可能会改变程序的执行结果。

as-if-serial 语义把单线程程序保护了起来，无需担心内存可见性问题。

### happens-before (先行发生) 原则
    
- 程序次序规则(Program Order Rule)：在一个线程内，按照程序代码顺序，书写在前面的操作先行发生于书写在后面的操作。
- 监视器锁规则（管程锁定规则）(Monitor Lock Rule)： 一个unlock操作在时间上先于后面对同一个锁的lock操作。
- volatile变量规则(volatile Variable Rule)： 对一个volatile变量的写操作在时间上先于后面对这个变量的读操作。
- 线程启动规则(Thread Start Rule)： Thread的start()方法先于这个线程的所有操作。
- 线程终止规则(Thread Termination Rule)： 线程中所有的操作都先于对这个线程进行终止检测的操作（Thread.join()、Thread.isAlive()）。
- 线程中断规则(Thread Interruption Rule)： Thread.interrupt() 方法调用先于这个线程检测到中断事件的发生。可以通过Thread.interruptted()检测是否有中断事件的发生。
- 对象终结规则(Finalizer Rule )： 一个对象的初始化先于它的finalize()方法；
- 传递性(Transitivity)：如果操作A先于操作B，操作B先于操作C，那么操作A先于操作C.
        

  时间先后顺序与先行发生原则之间基本没有太大的关系，衡量并发安全问题必须以先行发生（happens-before）原则。


## 顺序一致性模型（sequentially consistent）

 JMM保证正确同步的程序执行具有顺序一致性，即程序的执行结果与该程序的执行结果与该程序在程序一致性模型中的运行结果相同。
 
 在概念上，顺序一致性模型有一个单一的全局内存，这个内存通过一个左右摆动的开关可以连接到任意一个线程。
 
 ![](https://mynoteimage.oss-cn-beijing.aliyuncs.com/note/2021-09-12-075632.png)

### 特性
- 一个线程的所有操作必须按照程序顺序进行。    
- （不管程序是否同步）所有线程都只能看到单一的操作执行顺序。在顺序一致性内存模型中，每个操作都必须是原子执行且立刻对所有线程可见。
        

###  同步程序执行的顺序一致性效果
        
-   在JMM上运行的结果和在顺序一致性 模型中运行结果是一样的。虽然JMM会对程序进行重排序，但是由于线程（监视器）之间的互斥性，并不影响其他线程执行。 
        
### 未同步程序的执行特性
    
- JMM不保证未同步程序与该程序在顺序一致性模型中运行结果一致。
- 未同步的程序在JMM和顺序一致性模型上运行，整体上都是无序的，其执行结果也无法预知。
        
### 未同步的程序在两个模型上运行的差异 
- 顺序一致性模型保证单线程内操作按程序顺序执行，但是JMM不会。
- 顺序一致性模型保证所有线程 只能看到一致的操作执行顺序，但是JMM不会。
- 顺序一致性模型保证所有内存读、写操作都是原子性的，但是JMM对于long/double不保证是原子操作。（long/double会被拆分成两个32位数据操作）
            

## 锁
### 锁释放/获取的happens-before关系
- 根据程happens-before规则，线程A的代码（先获取到锁）happens before 于线程B的代码（后获取到表）
        
### 锁释放/获取的内存语义
释放锁时：JMM 会将线程工作内存中数据，刷新到主内存中。
获取锁时：JMM 会将工作内存中的数据失效，重新从主内存中获取副本。

- 线程A释放一个锁，实质上是线程A向接下来将要获取这个锁的某个线程发出了（线程A对共享变量所做修改的）消息。
- 线程B获取一个锁，实质上是线程B接收了之前某个线程发出的（在释放这个锁之前对共享变量所做修改的）消息。
- 线程A释放锁，随后线程B获取这个锁，这个过程实质上是线程A通过主内存向线程B发送消息。

> 释放锁和[[#volatile]]写有相同的语义
> 获取锁和[[#volatile]]读有相同的语义
        
### 锁内存语义的实现
    
- AQS：java同步器框架AbstractQueuedSynchronizer。AQS使用了一个整型的volatile变量（命名为state的）来维护同步状态。
- ReentrantLock 的公平锁和非公平锁
	- 公平锁和非公平锁释放锁时，最后都会写volatile的state变量。
	- 公平锁加锁时会读取这个state变量。
	- 非公平锁获取时，先用CAS（ compareAndSet()方法）更新state变量，这个操作同时具有volatile读和volatile写的内存语义。
- 锁释放-获取内存语义的实现方式
	- 利用volatile变量的读/写实现内存语义。
	- 利用CAS所附带的volatile读和volatile写的内存语义
            
	#### concurrent包的实现
   
	1.  声明共享变量为volatile；
        
    2.  使用CAS的原子条件更新来实现线程之间的同步；
        
    3.  配合以volatile的读/写和CAS所具有的volatile读和写的内存语义来实现线程之间的通信。
        

        AQS，非阻塞数据结构和原子变量类（java.util.concurrent.atomic包中的类），这些concurrent包中的基础类都是使用这种模式来实现的，而concurrent包中的高层类又是依赖于这些基础类来实现的。从整体来看，concurrent包的实现示意图如下：
		
		![](https://mynoteimage.oss-cn-beijing.aliyuncs.com/note/2021-09-04-F8317DF8-4E0E-492A-86A1-78CA285AF495.png)
		
		
		## JMM 的内存可见性保证
		Java 程序的内存可见性保证按程序类型可以分为下列三类：
		- 单线程程序，不会出现内存可见性问题。编译器，runtime，处理器会共同确保单线程程序的执行结果和在顺序一致性模型中的结果一致（具有顺序一致性）
		- 正确同步的多线程程序，具有顺序一致性。JMM 通过限制编译器和处理器的重排序
		- 未同步/未正确同步的多线程程序，提供了最小的安全保障，要么是之前某个线程写入的值，要么是默认值。（不可能是凭空出现的值）