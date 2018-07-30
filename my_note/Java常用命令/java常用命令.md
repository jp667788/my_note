- 详情参考：
    - [Java命令学习系列（零）——常见命令及Java Dump介绍](http://www.hollischuang.com/archives/308)
    - [javac - Java programming language compiler](https://docs.oracle.com/javase/7/docs/technotes/tools/windows/javac.html)
    - [javap - The Java Class File Disassembler](https://docs.oracle.com/javase/7/docs/technotes/tools/windows/javap.html)
    - [Java命令学习系列（一）——Jps-HollisChuang's Blog](http://www.hollischuang.com/archives/105)
    - [Java命令学习系列（二）——Jstack-HollisChuang's Blog](http://www.hollischuang.com/archives/110)
    - [Java命令学习系列（七）——javap-HollisChuang's Blog
](http://www.hollischuang.com/archives/1107)
    
- # javac
    描述：读取类和接口，使用java语言编写，并编译成calss字节码文件，它同样可以处理在java源代码和文件中的注释。
    使用语法： javac [options] [sourcefiles] [classes] \[@argfiles\] 参数的顺序任意
    
    参数的含义：

    - options：命令行选项
    - sourcesfiles：一个或多个源文件（如：MyClass.java）
    - classes：一个或多个需要处理注释的类（如：MyPackage.MyClass）  
    - @argfiles：一个或多个选项选项和源文件。 -J命令项

    在命令窗口输入'javac'命令，options有如下选择：
    
    ![](http://pbhc9u1ue.bkt.clouddn.com/javac.png)

    新建一个类TestA.java:
    ```
    public class TestA{
        public static void main(String[] args){

            int a = 1;
            int b = 2;
            int c = a + b;
            System.out.println("c:" + c);
        }
    }
    ```
    命令窗口执行：cd 文件目录 -> javac TestA.java
    TestA.java相同的目录中将会生成TestA.class文件，如图：
    ![](http://pbhc9u1ue.bkt.clouddn.com/javac2.png)

  

- # javap
    描述：反汇编class文件
    使用语法：javap [options] classes

    参数含义：

    - options：命令行选项
    - classes：一个或多个class文件，可以指定文件路径（如 C:\myproject\src\DocFooter.class）或URL（例如file:///C:/myproject/src/DocFooter.class）

    在命令行输入'javap'，options有如下选择：

    ![](http://pbhc9u1ue.bkt.clouddn.com/javap.png)
    
    javap根据options决定输出什么，如果没有使用options，javap就会输出包、类的protected和public域及类里的所有方法。

    使用 ‘javap’ 反编译刚刚的TestA.class:

    ![](http://pbhc9u1ue.bkt.clouddn.com/javap2.png)

    再加上一个命令项：-c
    
    ![](http://pbhc9u1ue.bkt.clouddn.com/javap3.png)

    其中有的翻译不太准确，见下：
    ```
    -help 帮助
    -l 输出行和变量的表
    -public 只输出public方法和域
    -protected 只输出public和protected类和成员
    -package 只输出包，public和protected类和成员，这是默认的
    -p -private 输出所有类和成员
    -s 输出内部类型签名
    -c 输出分解后的代码，例如，类中每一个方法内，包含java字节码的指令，
    -verbose 输出栈大小，方法参数的个数
    -constants 输出静态final常量
    ```

    这里可以看到一条条JVM指令，对栈的操作一目了然。

- # Jps
    jps是JDK1.5提供的一个**显示所有当前所有java进程pid**的命令

    jps可以显示当前运行的java进程以及相关参数，它的实现机制：
    java程序启动以后，会在指定目录下，就是临时文件生成一个hsperfdata_UserName的文件夹，这里的UserName值得是操作系统当前用户的用户名。这个文件位于：

    - Windows：C:\Users\{UserName}\AppData\Local\Temp\hsperfdata_{UserName}
    - Liunx:/tmp/hsperfdata_{UserName}/

    我开启了电脑中的项目，并打开了windows下的文件夹位置：

    显示有三个java进程正在执行

    ![](http://pbhc9u1ue.bkt.clouddn.com/jps1.png)

    再使用jps命令来校验一下：

    这里有四个进程，前面三个与上面文件夹中一致，多出一个是Jps命令自己的进程。

    ![](http://pbhc9u1ue.bkt.clouddn.com/jps2.png)
    

    如果jps -help 换弹出jps的使用语法：

    ![](http://pbhc9u1ue.bkt.clouddn.com/jps3.png)

    其中各个参数含义：

    为了查看这些参数首先定义一个类，执行无限循环：
    ```
    public class TestB{
        public static void main(String[] args) {
            while(true){
                System.out.println(1);
            }   
        }
    }
    ```
    - **-q** ：只显示pid，不显示class名称、jar文件名，和传递给main方法的参数。
    
        ![](http://pbhc9u1ue.bkt.clouddn.com/jps4.png)
    - **-m** ：输出传递给main方法的参数。在启动main方法时，传递两个参数：A和B：

        ![](http://pbhc9u1ue.bkt.clouddn.com/jps5.png)
    - **-v** ：输出传递给JVM的参数。在启动main方法时，给JVM传递一个参数：-Dfile.encoding=UTF-8：
     
        ![](http://pbhc9u1ue.bkt.clouddn.com/jps6.png)
    - **-l** ：输出应用程序main class的完整package名或者应用程序的jar文件完整路径。
       
       ![](http://pbhc9u1ue.bkt.clouddn.com/jps7.png)

- # 关于Java Dump
    Java Dump是Java虚拟机的运行时快照。将Java虚拟机运行时的状态和信息保存到文件。

    - 线程Dump，包含所有线程的运行状态。纯文本格式。
    - 堆Dump，包含线程Dump,包含所有堆对象的状态。二进制格式。

    Java Dump主要用于补足传统Bug分析手段的不足，可在任何Java环境使用，信息量充足。

    制作Java Dump：
    - 使用Java虚拟机制作Dump
    
        指示虚拟机在内存不足错误，自动生成Dump：-XX:+HeapDumpOnOutOfMemoryError

    - 使用图形化工具制作Dump
        
        使用JDK1.6自带工具：Java VisualVM
    - 使用命令行制作品Dump
        
        jstack:打印线程的栈信息,制作线程Dump。
        jmap:打印内存映射,制作堆Dump。

        步骤：

            1. 检查虚拟机版本（java -version）
            2. 找出目标Java应用的进程ID（jps）
            3. 使用jstack命令制作线程Dump • Linux环境下使用kill命令制作线程Dump
            4. 使用jmap命令制作堆Dump

- # Jstack
    - ### 功能
    
        描述：jstack用于生成java虚拟机当前时刻的线程快照。

        线程快照：当前java虚拟机内每一条线程正在执行的方法堆栈的集合。

        生成线程快照是为了定位线程出现停顿的原因，如线程死锁、死循环、请求外部资源能导致长时间等待等等。

        **jstack命令主要用来查看java线程的调用堆栈，可以用来分析线程问题**

    - ### 线程状态
        jstack命令查看线程堆栈信息可能会看到线程的几种状态：
        - NEW，未启动的。
        - RUNNABLE，在虚拟机内执行的。
        - BLOCKED，受阻塞并等待监视器锁。
        - WATING，无限等待另一个线程执行特定操作。
        - TIMED_WATING，有时限的等待另一个线程的特定操作。
        - TERMINATED，已退出的
    
    - ### 监视器（Monitor）
        **Monitor是Java中用以实现线程之间的互斥与协作的主要手段** ，每个对象都有且只有一个Monitor,下图描述了monitor与线程之间的关系：

        ![](http://pbhc9u1ue.bkt.clouddn.com/jstack1.png)

        **Entey Set 进入区**：表示线程通过synchronized要求获取对象的锁，如果对象未被锁，则进入The Owner，否则一直在进入区等待。

        **The Owner 拥有者**：表示某一线线程成功竞争到对象的锁。

        **Wait Set 等待区**：表示线程执行wait方法后，释放了线程的锁，并在等待区等待唤醒。

    - ### 调用修饰
        表示线程在方法调用时,额外的重要的操作。线程Dump分析的重要信息。修饰上方的方法调用。
        ```
        locked <地址> 目标：使用synchronized申请对象锁成功,监视器的拥有者。

        waiting to lock <地址> 目标：使用synchronized申请对象锁未成功,在迚入区等待。

        waiting on <地址> 目标：使用synchronized申请对象锁成功后,释放锁幵在等待区等待。

        parking to wait for <地址> 目标
        ```

    - ### 线程动作
    线程状态产生的原因：

        runnable:状态一般为RUNNABLE。

        in Object.wait():等待区等待,状态为WAITING或TIMED_WAITING。

        waiting for monitor entry:进入区等待,状态为BLOCKED。

        waiting on condition:等待区等待、被park。

        sleeping:休眠的线程,调用了Thread.sleep()。


    - ### 线程Dump的分析
        - 进入区等待
        ```
        "d&a-3588" daemon waiting for monitor entry [0x000000006e5d5000]
        java.lang.Thread.State: BLOCKED (on object monitor) at com.jiuqi.dna.bap.authority.service.UserService$LoginHandler.handle()
        - waiting to lock <0x0000000602f38e90> (a java.lang.Object)
        at com.jiuqi.dna.bap.authority.service.UserService$LoginHandler.handle()
        ```
        线程状态BLOCKED，线程动作wait on monitor entry,调用修饰waiting to lock总是一起出现，表示代码存在了冲突调用。

        - 同步阻塞
            
            一个线程锁住一个对象，大量其他线程在对象上等待。
        ```
        "blocker" runnable
        java.lang.Thread.State: RUNNABLE
        at com.jiuqi.hcl.javadump.Blocker$1.run(Blocker.java:23)
        - locked <0x00000000eb8eff68> (a java.lang.Object)
        "blockee-11" waiting for monitor entry
        java.lang.Thread.State: BLOCKED (on object monitor)
        at com.jiuqi.hcl.javadump.Blocker$2.run(Blocker.java:41)
        - waiting to lock <0x00000000eb8eff68> (a java.lang.Object)
        "blockee-86" waiting for monitor entry
        java.lang.Thread.State: BLOCKED (on object monitor)
        at com.jiuqi.hcl.javadump.Blocker$2.run(Blocker.java:41)
        - waiting to lock <0x00000000eb8eff68> (a java.lang.Object)
        ```
    **持续运行的IO**: IO操作是可以以RUNNABLE状态达成阻塞。例如:数据库死锁、网络读写。 格外注意对IO线程的真实状态的分析。 一般来说,被捕捉到RUNNABLE的IO调用,都是有问题的。

    - 总结：
        
        wait on monitor entry： 被阻塞的,肯定有问题

        runnable ： 注意IO线程

        in Object.wait()： 注意非线程池等待
    
    - ### 使用
    首先命令窗口中输入jstack -help

    ![](http://pbhc9u1ue.bkt.clouddn.com/jstack2.png)

    **-F** : 强制打印栈信息

    **-l** : 长列表，打印关于锁的附加信息。例如属于java.util.concurrent的ownable synchronizers列表

    **-m** : 打印java和native c/c++框架的所有栈信息.

    **-h** : 与 -help 一致，打印帮助信息

    首先，看之前一段死循环的代码，执行下面代码
    ```
    public class TestB{
        public static void main(String[] args) {
            while(true){
                System.out.println(1);
            }   
        }
    }
    ```

    首先使用jps查看进程号

    ![](http://pbhc9u1ue.bkt.clouddn.com/jstack3.png)

    再输出jstack 进程号 查看堆栈信息 ：

    ![](http://pbhc9u1ue.bkt.clouddn.com/jstack4.png)
    
    从上图中可以看出，当前有一条线程处于RUNNABLE状态，执行到了TestB的第4行。


    - ### 死锁分析
        **死锁**：是指两个或两个以上的进程在执行过程中，由于竞争资源或者由于彼此通信而造成的一种阻塞的现象，若无外力作用，它们都将无法推进下去。下面是一段死锁程序：
        ```
        public class JStackDemo {
            public static void main(String[] args) {
                Thread t1 = new Thread(new DeadLockclass(true));//建立一个线程
                Thread t2 = new Thread(new DeadLockclass(false));//建立另一个线程
                t1.start();//启动一个线程
                t2.start();//启动另一个线程
            }
        }
        class DeadLockclass implements Runnable {
            public boolean falg;// 控制线程
            DeadLockclass(boolean falg) {
                this.falg = falg;
            }
            public void run() {
                /**
                 + 如果falg的值为true则调用t1线程
                 */
                if (falg) {
                    while (true) {
                        synchronized (Suo.o1) {
                            System.out.println("o1 " + Thread.currentThread().getName());
                            synchronized (Suo.o2) {
                                System.out.println("o2 " + Thread.currentThread().getName());
                            }
                        }
                    }
                }
                /**
                 + 如果falg的值为false则调用t2线程
                 */
                else {
                    while (true) {
                        synchronized (Suo.o2) {
                            System.out.println("o2 " + Thread.currentThread().getName());
                            synchronized (Suo.o1) {
                                System.out.println("o1 " + Thread.currentThread().getName());
                            }
                        }
                    }
                }
            }
        }
        class Suo {
            static Object o1 = new Object();
            static Object o2 = new Object();
        }

        ```
        启动程序，控制台输出：

        ```
        o2 Thread-1
        o1 Thread-0
        ```
        但是程序并没有停止，仍在运行，此时已经差生了死锁。

        现在使用jstack查看堆栈信息：

        ![](http://pbhc9u1ue.bkt.clouddn.com/jstack5.png)

        打印的信息很明显的显示了：‘Found one Java-level deadlock’。


    - ### 其他
        虚拟机执行Full GC时,会阻塞所有的用户线程。因此,即时获取到同步锁的线程也有可能被阻塞。      