详情请参见：

- 《深入理解Java虚拟机 --周志明》 第6章 -- 类文件结构，第7章 -- 虚拟机类加载机制
- [http://www.hollischuang.com/archives/199](http://www.hollischuang.com/archives/199)
- [http://www.hollischuang.com/archives/201](http://www.hollischuang.com/archives/201)
- [http://www.importnew.com/24036.html](http://www.importnew.com/24036.html)
- 部分图片、描述来自[https://wx.zsxq.com/dweb-alpha/#/index/142111152552](https://wx.zsxq.com/dweb-alpha/#/index/142111152552)

# 1. 类加载机制
- ### 概述 
    
    虚拟机把描述类的数据从Class文件加载到内存，并对数据进行校验、转换解析和初始化，最终形成可以被虚拟机直接使用的java类型。

    类型的加载、连接和初始换的过程都是在程序运行期间完成的，虽然会稍微增加性能开销，但为java提供了高度的灵活性，使得java可以进行动态扩展。

- ### 类加载的时机
    类的生命周期：加载(loading) -> 验证(verification) -> 准备(Perparation) -> 解析(Resoloution) -> 初始化(Initialization) -> 使用(Using) -> 卸载(Unloading)


    ![avatar](http://pbhc9u1ue.bkt.clouddn.com/%E7%B1%BB%E7%9A%84%E7%94%9F%E5%91%BD%E5%91%A8%E6%9C%9F.jpg)

    其中，加载、验证、装备、初始化和卸载这5个阶段的顺序是确定的，但是解析阶段不一定。为了支持**Java语言运行时绑定**，在某些情况下可以在初始化阶段之后在开解析。

    虚拟即规范则是规定了**有且只有**5种情况必须立即对类进行"初始化":

    1. 遇到new、getstatic、putstatic、invokestatic这4条字节码指令事，如果类没有进行过初始化，需要先初始化。一般场景：使用new实例化对性、读取或设置静态字段（被final修饰、已经放入常量池中的静态字段）、调用一个类的静态方法。
    2. 进行反射调用时，如果类没有进行初始化，则需要先触发其初始化。
    3. 当初始化一个类的时候，如果发现其父类还没有进行过初始化，先初始化父类。
    4. 虚拟机启动会先初始化主类（包好main()方法的类）。
    5. 如果一个java.lang.invoke.MethodHandle实例最后的解析结果REF_getStatic、REF_putStatic、REF_invokeStatic 的方法句柄的类还没有初始化，先进行初始化。
    
    
# 2. 类加载的过程
- ### 加载

    “加载” 是 “类加载”过程中的一个阶段。虚拟机在这个阶段需要完成：

     - 通过一个类的全的全限定名来获取定义此类的二进制字节流。
     - 将这个字节流代表的静态存储结构转化为方法区的运行时数据结构
     - 在内存中生成一个代表这个类的java.lang.Class对象。

    全限定名：可以理解为类的绝对路径，一般为包名.外部类名$内部类名

    - 成员内部类：包名.外部类名$内部类名。
    - 匿名内部类：包名.外部类名$由1开始的正整数-按照类装载的顺序一次排列
    - 局部内部类：包名.外部类名$由1开始的正整数后跟局部类名-其中数字部分是局部类在外部类上下文出现的先后顺序。

- ### 验证
    验证是连接阶段的第一步，目的是为了确保CLass文件的字节流中包含的信息符合当前虚拟机的要求。 因为Class文件并不一定是Java源码编译的赖的，可以由任何途径产生，所以验证是虚拟机对自身保护。

    验包括下面4个阶段的检验：

    1. **文件格式验证**：验证字节流是否符合Class文件格式的规范，只有通过了这个阶段的验证，Class字节流才会进入方法区中进行存储。

    2. **元数据验证**：对字节码描述的信息进行语义分析，保证其描述的信息符合Java语言规范要求。

    3. **字节码验证**：通过数据流和控制流分析，确定程序语义是否合法、符合逻辑，这个阶段会对类的方法体进行校验分析，保证方法运行不会危害虚拟机安全。

    4. **符号引用验证**：发生在虚拟机将符号引用转化为直接引用的时候，这个转化动作发生在解析阶段中，对类自身以外（常量池中的各种符号引用）的信息进行匹配性校验。

- ### 准备
    准备阶段是正式为类变啦领分配内存并设置类变量初始值的阶段，这些变量的内存将在方法区中分配。需要注意的是：

    - 进行内存分配仅包括类变量(static变量)，不包括实例变量，实例变量在实例化时随对象一起分配在堆上。
    - 初始值通常情况下是数据类型的零值，真正的赋值阶段会在初始化阶段才会执行。

- ### 解析
    解析阶段是虚拟机将常量池内的符号引用替换为直接引用的过程。

    - 符号引用：以一组符号来描述所引用的目标。
    - 直接引用：可以是直接指向目标的指针、相对偏移量或是一个能间接定位到目标的句柄。
     
- ### 初始化
    类初始化是类加载过程的最后一步，到了初始化阶段，才开始真正执行类中定义的Java程序代码。在之前的“准备”阶段，变量已赋过了一次初始值。在初始化阶段，则根据程序制定的就好去初始化变量。 

    初始化阶段是执行类构造器的 &lt;clinit&gt;()方法的过程，关于&lt;clinit&gt;()方法，有下面几点：

    - &lt;clinit&gt;()方法由比编译器收集类中的所有类变量的赋值动作和stattic{}块中的语句合并产生，编译器收集的顺序由语句在源文件的顺序决定，**静态块中只能访问到定义在静态语句块之前的变量，定义在他之后的变量，在前面的静态块中只能赋值，但不能访问。**如下代码：
        ```
            public class Test{
                static{
                    i = 0;
                    System.out.println(i); // 编译器提示“非法的向前引用”
                }
                static int i = 1;
            }
        ```


    - &lt;clinit&gt;()方法不需要显示调用父类构造器，在执行子类的&lt;clinit&gt;()方法之前，会保证先完成父类的&lt;clinit&gt;()方法。所以第一个被执行的&lt;clinit&gt;()方法的类肯定是Object。
    
    -  由于父类的&lt;clinit&gt;()方法优先于子类的执行，所以父类的静态块优先于子类的静态块执行。

    - &lt;clinit&gt;()方法对于类或者接口来说并不是必需的，如果没有静态块也没有对变量赋值，编译器就不会为这个类生成&lt;clinit&gt;()方法。

    - 接口中不能使用静态块，但是可以进行变量初始化的操作，不过，**执行接口的&lt;clinit&gt;()方法不需要先直接父接口的&lt;clinit&gt;()方法，接口的实现类初始化时也不会执行接口的&lt;clinit&gt;()方法**

    - 虚拟机保证一个类的&lt;clinit&gt;()方法多线程中被正确的加锁、同步，当一个线程执行完&lt;clinit&gt;()方法之后，别的线程不会再去执行&lt;clinit&gt;()方法，**同一个类加载器下，一个类只会初始化一次。**

# 3. 类加载器

类加载器通过一个类的全限定名来获取描述此类的二进制字节流，以便让程序自己决定如何去获取所需的类。

对于任意一个类，都需要由加载它的类加载器和这个类本身一起确定其在虚拟机中的唯一性。也就是说，即使两个类来源于同一个Class文件，被同一个虚拟机加载，但是如果加载它们的类加载器不同，那着两个类就不会相等。

- ### 双亲委派模型
    
    类加载器分为三种：
    - 启动类加载器：加载<JAVA_HOME>\lib目录中的类库
    - 扩展类加载器：加载<JAVA_HOME>\lib\ext目录中的类库
    - 应用程序类加载器：加载ClassPath中指定的类库
    

    类加载器双亲委派模型如下图所示

    ![](http://pbhc9u1ue.bkt.clouddn.com/%E7%B1%BB%E5%8A%A0%E8%BD%BD%E5%99%A8.jpg)

    双亲委派模型要求除了顶层的启动类加载器外，其余的类加载器都应当有自己的父类加载器。

    双亲委派模型工作流程：
    - 类加载器收到了类加载的请求
    - 请求委派给父类加载器去完成
    - 当父类加载器无法进行加载，子加载器才会尝试自己加载

    使用双亲委派模型是的Java中的类具有了带有优先级的层次关系，所以，及时我们自定义了一个与rt.jar类库中的类重名的类，可以正常编译，但是无法运行加载。

- ### 双亲委派模型的实现

    java.lang.ClassLoader的loadClass方法中，就是双亲委派的实现代码：

    ``` java
    protected Class&lt;?> loadClass(String name, boolean resolve)
        throws ClassNotFoundException{
        synchronized (getClassLoadingLock(name)) {
            // 首先，检查请求的类是否已经被加载过了
            Class c = findLoadedClass(name);
            if (c == null) {
                long t0 = System.nanoTime();
                try {
                    if (parent != null) {
                        c = parent.loadClass(name, false);
                    } else {
                        c = findBootstrapClassOrNull(name);
                    }
                } catch (ClassNotFoundException e) {
                    // 如果父类加载器抛出ClassNotFoundException
                    // 说明父类加载器无法完成加载请求
                }
                if (c == null) {
                    // 在父类加载器无法加载的时候
                    // 在调用本身的findClass方法进行类加载
                    long t1 = System.nanoTime();
                    c = findClass(name);
                    // this is the defining class loader; record the stats
                    sun.misc.PerfCounter.getParentDelegationTime().addTime(t1 - t0);
                    sun.misc.PerfCounter.getFindClassTime().addElapsedTimeFrom(t1);
                    sun.misc.PerfCounter.getFindClasses().increment();
                }
            }
            if (resolve) {
                resolveClass(c);
            }
            return c;
        }
    }
    ```

双亲委派模型的方法调用过程如下图：

![](http://pbhc9u1ue.bkt.clouddn.com/类加载器调用.jpg)
    
- ### 破坏双亲委派模型
    **重写 loadClass() 方法**
	
	- loadClass(): 进行类加载，双亲委派的实现。
	- findClass()：根据名称或位置加载 .class 字节码。在 loadClass 中被调用，ClassLoader 默认不实现，直接抛出异常
	- defineClass()：把字节码转化成 class

对于自定义加载器：
- 重写 loadClass 方法，打破双亲委派
- 只重写 findClass 方法，不打破双亲委派

#### loadClass 和 findClass 的区别
loadClass 是 JDK 双亲委派的实现，findClass 是子类实现的方法。loadClass 中，在双亲委派模式加载失败后，才执行 findClass 方法。
JDK 推荐遵循双亲委派，所以通过提出 findClass 方法，方便重写。

	

