### JVM内存结构

- 堆(Heap)
- 方法区(Method Area)
- 程序计数器(Program Counter Register)
- 本地方法栈(Native Method Stack)
- 虚拟机栈(VM Stack)

![](https://mynoteimage.oss-cn-beijing.aliyuncs.com/note/2022-03-17-Image.png)

#### 堆(Heap)
- 堆是Java虚拟机中内存最大的一块区域
- 堆是被所有线程共享的一块区域
- 所有的对象实例及数组都要在堆上分配(但是对着JIT编译器的发展和逃逸分析算法的成熟，所有的对象实例及数组都在堆上分配这一点并不是那么绝对，关于这里的更多描述请看这里[http://www.hollischuang.com/archives/2398](http://www.hollischuang.com/archives/2398))。

java堆中分为新生代和老年代。新生代中可以再细分为：伊甸园(Eden)、幸存1区(From Survivor)、幸存2区(To Survivor)。

![](https://mynoteimage.oss-cn-beijing.aliyuncs.com/note/2022-03-17-A2217599-BB72-4C47-8945-7816F4725A56.png)

java堆在物理上不要求一定是连续的内存空间，只要逻辑上连续即可。既可以固定大小，也可以拓展。（通过-Xmx和-Xms控制）


#### 程序计数器(Program Counter Register)
程序计数器在JVM的空间较小，而且它是**线程私有**的。它可以看做是当前线程所执行的字节码行号指令器。正因为有了程序计数器，在多线程进行线程切换时，线程能够记住执行到了哪里。程序计数器也是JVM中唯一没有规定(不会抛出)OutOfMemoryError的情况的区域

#### 虚拟机栈(VM Stack)

虚拟机栈在JVM中也是**线程私有**的，它的生命周期与线程相同。每个方法执行的时候会创建一个栈帧(Stack Frame)，用于存储:
- 局部变量表(Local Variable Table)
	- 存放对象：
		- 编译期可知的各种基本数据类型
		- 对象引用 
			- reference类型
			- 可能是一个指向对象起始地址的指针
			- 也可能是指向代表对象的句柄
		- returnAddress 类型
			- 指向了一条字节码指令的地址
	- 存储空间：局部变量槽
		- 64位的 long 、double 占用两个槽
		- 其余类型1个
	- 内存空间在编译期完成分配
	- 进入一个方法时，方法在栈帧中需要的局部变量表空间确定
	- 并且运行期间不会改变
- 操作数栈(Operand Stack)
- 动态连接(Dynamic Linking)
- 方法出口等信息

每个方法从开始调用到执行完成的过，就是对应的栈帧从入栈到出栈的过程。(更多可见https://iamjohnnyzhuang.github.io/java/2016/07/12/Java%E5%A0%86%E5%92%8C%E6%A0%88%E7%9C%8B%E8%BF%99%E7%AF%87%E5%B0%B1%E5%A4%9F.html)

 其中，局部变量表中存放了基本数据类型，和引用类型的对象指针和一套字节码指令的地址。其中long和double会占用两个局部变量的空间。局部变量的空间在编译期就完成了分配，并切不会在运行期间发生改变。

 虚拟机栈可能会发生一下两种异常：

 1. StackOverflowError：线程请求的栈深度大于虚拟机所允许的深度。
 2. OutMermoryError：栈扩展是无法再申请到内存。 

#### 本地方法栈(Native Method Stack)

 本地方法栈与虚拟机栈非常的相似，本地方法栈为虚拟机使用的Native方法服务。本地方法栈可以存放别的语言的方法。

#### 方法区(Method Area)

- 方法区也被所有的线程共享
- 用于存储对象的类型信息，这些类型信息包括：
	- 虚拟机已经加载的类的信息
	- 常量
	- 静态变量

- JDK 1.7 之前
	- 方法区由永久代实现


#### 运行期常量池(Runtime Costant Pool)
方法区中还有一部分区域是运行时常量池(Runtime Costant Pool)，用于存放编译期生成的各种字面量和符号引用，这些内容在完成类的加载后进入运行事常量池存放。

Class 文件中除了有类的 版本、字段、方法、接口等描述信息外，还有一个**常量池表(Constant Pool Table)**

但是常量并不是一定在编译期才能放入常量池中，在运行期产生的常量也可以放入常量池中，比如使用String的intern()方法（更多请见：https://mritd.com/2016/03/22/java-memory-method-area-and-runtime-constant-pool)

![](https://mynoteimage.oss-cn-beijing.aliyuncs.com/note/2022-03-17-7140D3B7-2B30-4FC9-A890-A98E368E5765.png)

#### 直接内存(Direct Memory)

 直接内存并不是JVM内存中的一部分，但是这一部分也会导致OutOfMemoryError异常。