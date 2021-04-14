# java编译与反编译
- 详情参照：
    - [Java代码的编译与反编译那些事儿-HollisChuang's Blog](http://www.hollischuang.com/archives/58)
    - [我反编译了Java 10的本地变量类型推断-HollisChuang's Blog](http://www.hollischuang.com/archives/2187)
    - [Java命令学习系列（七）——javap-HollisChuang's Blog](http://www.hollischuang.com/archives/1107)
    - [Java中的Switch对整型、字符型、字符串型的具体实现细节](http://www.hollischuang.com/archives/61)

   ## 什么是编译
   
   将便于人编写、阅读、维护的高级计算机语言所写作的源代码程序，翻译为计算机能解读、运行的低阶机器语言的程序的过程就是编译。


   java中负责编译的编译器的命令：javac
    > javac是收录于JDK中的Java语言编译器。该工具可以将后缀名为.java的源文件编译为后缀名为.class的可以运行于Java虚拟机的字节码。 


    ## 什么是反编译
    反编译的过程与编译刚好相反，就是将已编译好的编程语言还原到未编译的状态，也就是找出程序语言的源代码。 

    ### Java反编译工具
    - javap
    
    javap是jdk自带的编译器。javap生成的文件并不是java文件，并不是那么容易理解。。

    新建测试类JavapTest.java，如下：

    ```
        public class JavapTest {
            public static void main(String[] args) {
                String str = "world";
                switch (str) {
                    case "hello":
                        System.out.println("hello");
                        break;
                    case "world":
                        System.out.println("world");
                        break;
                    default:
                        break;
                }
            }
        }
    ```

    执行:

    ```
        javac JavapTest.java
        javap -c JavapTest.class
    ```

    生成信息如下：

    ```
        Compiled from "JavapTest.java"
        public class JavapTest {
          public JavapTest();
        Code:
           0: aload_0
           1: invokespecial #1                  // Method java/lang/Object."<init>":()V
           4: return
          public static void main(java.lang.String[]);
        Code:
           0: ldc           #2                  // String world
           2: astore_1
           3: aload_1
           4: astore_2
           5: iconst_m1
           6: istore_3
           7: aload_2
           8: invokevirtual #3                  // Method java/lang/String.hashCode:()I
          11: lookupswitch  { // 2
                  99162322: 36
                 113318802: 50
                   default: 61
              }
          36: aload_2
          37: ldc           #4                  // String hello
          39: invokevirtual #5                  // Method java/lang/String.equals:(Ljava/lang/Object;)Z
          42: ifeq          61
          45: iconst_0
          46: istore_3
          47: goto          61
          50: aload_2
          51: ldc           #2                  // String world
          53: invokevirtual #5                  // Method java/lang/String.equals:(Ljava/lang/Object;)Z
          56: ifeq          61
          59: iconst_1
          60: istore_3
          61: iload_3
          62: lookupswitch  { // 2
                         0: 88
                         1: 99
                   default: 110
              }
          88: getstatic     #6                  // Field java/lang/System.out:Ljava/io/PrintStream;
          91: ldc           #4                  // String hello
          93: invokevirtual #7                  // Method java/io/PrintStream.println:(Ljava/lang/String;)V
          96: goto          110
          99: getstatic     #6                  // Field java/lang/System.out:Ljava/io/PrintStream;
         102: ldc           #2                  // String world
         104: invokevirtual #7                  // Method java/io/PrintStream.println:(Ljava/lang/String;)V
         107: goto          110
         110: return
    }
    ```

    javap 反编译并不没有直接生成了java文件，而是类似于字节码文件，这些字节码我们可以看得懂。查看上面的字节码文件，可以发现switch就是把字符串转成了hashCode在进行比较。

    - jad
    
    使用jad命令进行反编译，jad安装请看这里：https://www.jianshu.com/p/df8323916b26

    输入命令：`jad JavapTest.class`，生成信息如下：

    ```
    // Decompiled by Jad v1.5.8g. Copyright 2001 Pavel Kouznetsov.
    // Jad home page: http://www.kpdus.com/jad.html
    // Decompiler options: packimports(3) 
    // Source File Name:   JavapTest.java
    import java.io.PrintStream;
    public class JavapTest
    {
        public JavapTest()
        {
        }
        public static void main(String args[])
        {
            String s = "world";
            String s1 = s;
            byte byte0 = -1;
            switch(s1.hashCode())
            {
            case 99162322: 
                if(s1.equals("hello"))
                    byte0 = 0;
                break;
            case 113318802: 
                if(s1.equals("world"))
                    byte0 = 1;
                break;
            }
            switch(byte0)
            {
            case 0: // '\0'
                System.out.println("hello");
                break;
            case 1: // '\001'
                System.out.println("world");
                break;
            }
        }
    }

    ```

    这个就很清楚的可以看到原来字符串的switch是通过equals()和hashCode()方法来实现的。

    - CFR
    
    jad很好用，但是无奈的是很久没更新了，所以只能用一款新的工具替代他，CFR是一个不错的选择

    输入命令：`java -jar cfr_0_125.jar switchDemoString.class --decodestringswitch false`，得到如下信息:


    ```
    /*                                                          
     * Decompiled with CFR 0_131.
     */
    import java.io.PrintStream;
    public class JavapTest {
        public static void main(String[] arrstring) {
            String string;
            String string2 = string = "world";
            int n = -1;
            switch (string2.hashCode()) {
                case 99162322: {
                    if (!string2.equals("hello")) break;
                    n = 0;
                    break;
                }
                case 113318802: {
                    if (!string2.equals("world")) break;
                    n = 1;
                }
            }
            switch (n) {
                case 0: {
                    System.out.println("hello");
                    break;
                }
                case 1: {
                    System.out.println("world");
                    break;
                }
            }
        }
    }
    ```

    通过这段代码也能得到字符串的switch是通过equals()和hashCode()方法来实现的结论。


    ## 如何防止反编译

    典型的应对策略有以下几种：
    - 隔离Java程序        
        
        让用户接触不到你的Class文件
    - 对Class文件进行加密

        提到破解难度
    - 代码混淆
    
        将代码转换成功能上等价，但是难于阅读和理解的形式


    ## 利用反编译，探索Java10的本地变量类型推断
    本地变量类型推断是一个新的语法糖，本地变量类型推断将引入“var”关键字，而不需要显式的规范变量的类型。

    新建java文件，如下：
    ```
    public class VarDemo {
        public static void main(String[] args) {
            //初始化局部变量  
            var string = "hollis";
            //初始化局部变量  
            var stringList = new ArrayList<String>();
            stringList.add("hollis");
            stringList.add("chuang");
            stringList.add("weChat:hollis");
            stringList.add("blog:http://www.hollischuang.com");
            //增强for循环的索引
            for (var s : stringList){
                System.out.println(s);
            }
            //传统for循环的局部变量定义
            for (var i = 0; i < stringList.size(); i++){
                System.out.println(stringList.get(i));
            }
        }
    }
    ```

    使用java10的编译命令进行编译，得到如下信息：
    ```
    public class VarDemo {
        public static void main(String args[])
        {
            String s = "hollis";
            ArrayList arraylist = new ArrayList();
            arraylist.add("hollis");
            arraylist.add("chuang");
            arraylist.add("weChat:hollis");
            arraylist.add("blog:http://www.hollischuang.com");
            String s1;
            for(Iterator iterator = arraylist.iterator(); iterator.hasNext(); System.out.println(s1))
                s1 = (String)iterator.next();
            for(int i = 0; i < arraylist.size(); i++)
                System.out.println((String)arraylist.get(i));
        }
    }
    ```

    所以，本地变量类型推断，也是Java10提供给开发者的语法糖。虽然我们在代码中使用var进行了定义，但是对于虚拟机来说他是不认识这个var的，在java文件编译成class文件的过程中，会进行解糖，使用变量真正的类型来替代var（如使用String string 来替换 var string）。对于虚拟机来说，完全不需要对var做任何兼容性改变，因为他的生命周期在编译阶段就结束了。唯一变化的是编译器在编译过程中需要多增加一个关于var的解糖操作。

    虽然有了本地类型腿短，但是java依然是强类型语言，而不是与javaScript这种弱类型语言先相同。