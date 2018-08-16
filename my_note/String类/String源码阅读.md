# String源码阅读
详情参考：

- [Java 7 源码学习系列（一）——String](http://www.hollischuang.com/archives/99) 
- [Java中由substring方法引发的内存泄漏](https://blog.csdn.net/diaorenxiang/article/details/39155237)


    ## 一、 概述
    String 是Java中非常基础和重要的类，Stirng是典型的Immutable类，即不可变类。（如果一个对象它被构造后其，状态不能改变，则这个对象被认为是不可变的（immutable ））。String声明为final class，所有属性也是final，这同时也意味着String是**无法继承的**。

    Java语言提供了对字符串连接运算符的特别支持(+)，+ 号也可以将其他类型转成字符串，通过对象的toString方法实现。由于String是不可变的，所以String在进行拼接、裁剪等字符串操作时，都会产生新的String对象。

    Java中还提供了StringBuffer、StringBuilder类，来更好地解决String拼接而产生新对象的问题。

    ## 二、Stirng源码

    ### 1. 定义
    
    进入java.lang.String下，可以看到String类如下定义：

    `   public final class String implements java.io.Serializable, Comparable<String>, CharSequence`

    可以清楚的看到，String类被声明为final类，且实现了Serializable、Comparable、CharSequence 接口。其中CharSequencetigon 接口中提供了length()、chatAt() 等方法。

    ### 2. 属性
    ```
        private final char value[];（JDK 1.8）    
        private final byte value[];（JDK 1.9）
    ```

    value[]数组用于存储String中的字符串内容。是一个被声明成final的字符数组，在JDK1.9以后，value[]被声明为字节数组。因为是被final声明的，所以String一旦被初始化之后，就允许再改变。

    ```
        private int hash;
    ```

    hash 缓存了字符串的hashCode值，默认为0

    ```
    private static final long serialVersionUID = -6849794470754667710L;
    private static final ObjectStreamField[] serialPersistentFields = new ObjectStreamField[0];
    ```

    String实现了 Serializable 接口，所以支持序列化和反序列化。

    >   Java的序列化机制是通过在运行时判断类的serialVersionUID来验证版本一致性的。在进行反序列化时，JVM会把传来的字节流中的serialVersionUID与本地相应实体（类）的serialVersionUID进行比较，如果相同就认为是一致的，可以进行反序列化，否则就会出现序列化版本不一致的异常(InvalidCastException)。 

    JDK1.9中新增了一个coder属性：
    
    ```
    private final byte coder;
    ```
    >   此属性为用于编码字节的编码的标识符，分为 LATIN1 与 UTF16，被虚拟机信任，不可变，不重写。
       
    ### 3. 构造方法
    String类中包含了许多的构造方法（去除废弃的有13个），这里介绍几个常用的构造方法。
    
    - **String() -- 空构造**

        ```
        public String() {       
        this.value = "".value;
        }
        ```
    
        可以看到调用空构造时，会创建一个空字符对象。

    - **String(String original) -- 使用字符串创建一个字符串对象**

    ```
        public String(String original) {
            this.value = original.value;
            this.hash = original.hash;
        }
    ```

    与直接用""双引号创建字符串不同的是，使用new String("")创建字符串时，每个创建出来的对象都是存储在堆上的新对象，而使用""双引号创建出来的字符串从常量池中获取。所以出现如下代码中的情况：

    ```
        public class TestStringCons{
            public static void main(String[] args){
                String abc = "abc";
                String abc2 = new String("abc");
                String abc3 = new String("abc");
                String abc4 = "abc";
                System.out.println(abc == abc2); // false
                System.out.println(abc2 == abc3); // false
                System.out.println(abc == abc4); // true
            }
        }
    ```

    -  **String（Char[] value[]），String(char value[], int offset, int count)  -- 使用字符数组创建对象**
    
    ```
        public String(char value[]) {
            this.value = Arrays.copyOf(value, value.length);
        }      
        public String(char value[], int offset, int count) {
            if (offset < 0) {
                throw new StringIndexOutOfBoundsException(offset);
            }
            if (count <= 0) {
                if (count < 0) {
                    throw new StringIndexOutOfBoundsException(count);
                }
                if (offset <= value.length) {
                    this.value = "".value;
                    return;
                }
            }
            // Note: offset or count might be near -1>>>1.
            if (offset > value.length - count) {
                throw new StringIndexOutOfBoundsException(offset + count);
            }
            this.value = Arrays.copyOfRange(value, offset, offset+count);
        }
    ```

    传入字符数组创建时，会用到Arrays.copyOf方法和Arrays.copyOfRange方法。这两个方法是将原有的字符数组中的内容逐一的复制到String中的字符数组中。

    
    - **String(byte bytes[], int offset, int length, Charset charset)** -- 使用字节数组创建对象
    
    ```
        public String(byte bytes[], int offset, int length, Charset charset) {
            if (charset == null)
                throw new NullPointerException("charset");
            checkBounds(bytes, offset, length);
            this.value =  StringCoding.decode(charset, bytes, offset, length);
        }
    ```

    在Java中，String实例中保存有一个char[]字符数组，char[]字符数组是以unicode码来存储的，String 和 char 为内存形式，byte是网络传输或存储的序列化形式。所以在很多传输和存储的过程中需要将byte[]数组和String进行相互转化。所以，String提供了一系列重载的构造方法来将一个字符数组转化成String，提到byte[]和String之间的相互转换就不得不关注编码问题。**通过charset来解码指定的byte数组，将其解码成unicode的char[]数组，够造成新的String。**

    > 这里的bytes字节流是使用charset进行编码的，想要将他转换成unicode的char[]数组，而又保证不出现乱码，那就要指定其解码方式
    
    如果我们在使用byte[]构造String的时候，使用的是下面这四种构造方法(带有charsetName或者charset参数)的一种的话，那么就会使用StringCoding.decode方法进行解码，使用的解码的字符集就是我们指定的charsetName或者charset。 我们在使用byte[]构造String的时候，如果没有指明解码使用的字符集的话，那么StringCoding的decode方法首先调用系统的默认编码格式，如果没有指定编码格式则默认使用ISO-8859-1编码格式进行编码操作。主要体现代码如下：

    ```
          static char[] decode(byte[] ba, int off, int len) {
            String csn = Charset.defaultCharset().name();
            try {
                // use charset name decode() variant which provides caching.
                return decode(csn, ba, off, len);
            } catch (UnsupportedEncodingException x) {
                warnUnsupportedCharset(csn);
            }
            try {
                return decode("ISO-8859-1", ba, off, len);
            } catch (UnsupportedEncodingException x) {
                // If this code is hit during VM initialization, MessageUtils is
                // the only way we will be able to get any kind of error message.
                MessageUtils.err("ISO-8859-1 charset not available: "
                                 * x.toString());
                // If we can not find ISO-8859-1 (a required encoding) then things
                // are seriously wrong with the installation.
                System.exit(1);
                return null;
            }
        }
    ```

    在JDK1.9中，这个构造方法和StringCoding.decode方法发生一些改变：
    ```
         public String(byte bytes[], int offset, int length, Charset charset) {
            if (charset == null)
                throw new NullPointerException("charset");
            checkBoundsOffCount(offset, length, bytes.length);
            StringCoding.Result ret =
                StringCoding.decode(charset, bytes, offset, length);
            this.value = ret.value;
            this.coder = ret.coder;
        }
    ```

    其中StringCoding.decode在JDK1.9中不再返回char[]数组，而返回的是 StringCodingde 静态内部类 Result，再将Result中的value和coder赋给String。

    - **String(StringBuffer buffer)、String(StringBuilder builder) -- 使用StringBuffer、StringBuilder创建字符串**

    ```
        public String(StringBuffer buffer) {
            synchronized(buffer) {
                this.value = Arrays.copyOf(buffer.getValue(), buffer.length());
            }
        }
        public String(StringBuilder builder) {
            this.value = Arrays.copyOf(builder.getValue(), builder.length());
        }
    ```

    - **一个特殊的保护(protected)类型的构造方法**
    
        String中提供了一个protected修饰的构造器：

    ```
        String(char[] value, boolean share) {
            // assert share : "unshared not supported";
            this.value = value;
        }
    ```

    该方法与String(char[] value)的区别是：第一，多了一个boolean类型的share参数。这个参数方法中并没有用到，其实**加入这个boolean参数share是为了和String(cahr[] value) 这个构造器区分开来。**第二，这个构造器将传入的字符数组直接赋给了value 。而String(char[] value) 这个构造器将传入的字符数组使用Arrays.copyOf()方法复制了一份。

    > 使用该构造器的优点：**性能好**。不需要复制数组；节约内存，因为共享同一个数组，所以不需要新建数组空间。
    
    之所以这个构造方法被设置为poretected，如果设置为public，就有可能破坏String的不可变性。所以，从安全角度来看，这个构造器也是安全的。

    String的一些方法也使用了这种"性能好、节约内存、安全的构造器"，比如replace、concat、valueOf()以及JDK1.6的substring方法(实际上他们使用的是public String(char[], int, int)方法，原理和本方法相同，已经被本方法取代)。

    
    ### 4.substring
    
    substring 方法的作用就是提取某个字符串的子串。但是JDK6的substring 可能会导致内存泄露。先看一下JDK1.6 substring 的源码：
    ```
        public String substring(int beginIndex, int endIndex) {
            if (beginIndex < 0) {
                throw new StringIndexOutOfBoundsException(beginIndex);
            }
            if (endIndex > count) {
                throw new StringIndexOutOfBoundsException(endIndex);
            }
            if (beginIndex > endIndex) {
                throw new StringIndexOutOfBoundsException(endIndex - beginIndex);
            }
            return ((beginIndex == 0) && (endIndex == count)) ? this :
                new String(offset + beginIndex, endIndex - beginIndex, value); //使用的是和父字符串同一个char数组value
            }

        // 没有新差创建对象，仍然使用了原字符串对象
        String(int offset, int count, char value[]) {
            this.value = value;
            this.offset = offset;
            this.count = count;
        }
    ```

    由于返回回来的子字符串和原有的父字符串是同一个对象，就可能引发内存泄露：

    ```
        String str = "abcdefghijklmnopqrst";
        String sub = str.substring(1, 3) + "";
        str = null;    
    ```

    上面代码中，虽然str = nulln，但是sub依然引用了str所引用的对象，导致str 所指向的对象 "abcdefghijklmnopqrst" 无法被回收，进而可能导致内存泄露。

    为了改正这个问题，JDK1.7 之后的 substring 方法进行了修改，下面是JDK1.7的 substring 方法源码：
    ```
        public String substring(int beginIndex, int endIndex) {
            if (beginIndex < 0) {
                throw new StringIndexOutOfBoundsException(beginIndex);
            }
            if (endIndex > value.length) {
                throw new StringIndexOutOfBoundsException(endIndex);
            }
            int subLen = endIndex - beginIndex;
            if (subLen < 0) {
                throw new StringIndexOutOfBoundsException(subLen);
            }
            return ((beginIndex == 0) && (endIndex == value.length)) ? this
                    : new String(value, beginIndex, subLen);
        }


    public String(char value[], int offset, int count) {
        if (offset < 0) {
            throw new StringIndexOutOfBoundsException(offset);
        }
        if (count < 0) {
            throw new StringIndexOutOfBoundsException(count);
        }
        // Note: offset or count might be near -1>>>1.
        if (offset > value.length - count) {
            throw new StringIndexOutOfBoundsException(offset + count);
        }
        this.value = Arrays.copyOfRange(value, offset, offset+count);
    }

    public static char[] copyOfRange(char[] original, int from, int to) {
        int newLength = to - from;
        if (newLength < 0)
            throw new IllegalArgumentException(from + " > " + to);
        char[] copy = new char[newLength];   //是创建了一个新的char数组
        System.arraycopy(original, from, copy, 0,
                         Math.min(original.length - from, newLength));
        return copy;
    }

    ```

    可以发现是去为子字符串创建了一个新的char数组去存储子字符串中的字符。这样子字符串和父字符串也就没有什么必然的联系了，当父字符串的引用失效的时候，GC就会适时的回收父字符串占用的内存空间。

    ###5. String对 '+' 的重载

    Java是不支持运算符重载的，String 的 '+' 是 java 中唯一的一个重载运算符。先看下面一段代码 
    ```
    public class TestA{
      public static void main(String[] args){
        String str1 = "Hello";
        String str2 = str1 + "World";
      }
    }
    ```
    
    反编译上面的代码：

    ```
    public class TestA{
        public static void main(final String[] array) {
            new StringBuilder().append("Hello").append("World").toString();
        }
    }
    ```
    
    可以看出，String 中对 '+' 的重载其实就是使用StringBuilder 和 toString() 方法进行处理。

    ###6. Stirnrg.valueOf() 和 Integer.toString的区别

    ```
        1.int i = 5;
        2.String i1 = "" + i;
        3.String i2 = String.valueOf(i);
        4.String i3 = Integer.toString(i);
    ```

    第3行和第4行没有什么区别，因为String.valueOf(i) 也是调用了 Integer.toString()方法来实现的。

    第2行代码其实是String i1 = (new StringBuilder()).append(i).toString()。首先创建了一个StringBuilder 对象，在讲
    
    ###7. intern() 方法

    ![](http://pbhc9u1ue.bkt.clouddn.com/intern.png)

    intern() 方法有两个作用：

    - 第一，如果常量池中没有该字符串的字面量，将字符串字面量放入常量池。
    - 第二，返回这个常量的引用。

    首先看下面一段代码：
    ```
        String str1 = "Hello";
        String str2 = new String("Hello");
        String str3 = new String("Hello").intern();
        System.out.println(str1 == str2); // false
        System.out.println(str1 == str3); // true
    ```

    首先需要了解几个关键词：
    - 运行时常量池
        JVM 中有几种常量池：
        - class文件中的常量池
            
            > 主要用于存放**字面量**和**符号引用**，这部分内容会在类加载之后进入方法区与运行时常量池存放。
        - 方法区中的运行时常量池
            > 运行时常量池除了存放calss文件常量池的内容外，与class常量池不同的是，运行时茶凉吃具有动态性，在运行期也可能将新的常量放入池中。
        
        JVM为了减少JVM中创建的字符串数量，字符串类维护了一个常量池，主要用来存储编译期生成的各种字面量和符号引用。
    - 字面量
        > 如文本字符串、声明为final 的常量值等;
    - 符号引用
        >1.类和接口的全限定名；2.字段名称和描述符；3.方法名称和描述符。
    
    对于上面的代码产生的结果，先分析 new String("Hello") 创建对象的过程

    首先，编译期间，符号引用 str1 和字面量 Hello 会被加入到class文件中的常量池中，在类加载之后(具体时间请参考：[Java 中new String("字面量") 中 "字面量" 是何时进入字符串常量池的?](https://www.zhihu.com/question/55994121))
    但是并不是所有的字面量都会进入字符串常量池，如果字符串已经存在常量池中就不会再加载进来了。

    到了运行时期，执行到 new String("Hello")时，会在Java堆中创建一个字符串对象，这个对象所对应的字符串字面量保存在常量池中，但是 符号引用 Str1 指向的是堆中新创建出来的地址。所以会有以下代码成立：

    ```
    Stirng s1 = new String("Hello");
    Stirng s2 = new String("Hello");
    System.out.println(s1 == s2); // false
    ```

    因为s1,s2是堆上两个不同对象的地址引用，所以s1 == s2 为false。内存结构图大致如下图（草图）所示：
    
    ![](http://pbhc9u1ue.bkt.clouddn.com/String1.jpg)

    > 在不同版本的JDK中，Java堆和字符串常量池之间的关系也是不同的，这里为了方便表述，就画成两个独立的物理区域了。
    
    #### new String("Hello")创建了几个对象？

    所以可以很清楚的看到在执行 new String("Hello") 一共创建了两个对象，一个是s1所引用的堆空间中的对象，另一个是在常量池中的对象。

    JVM并没有规定常量池中的对象必须在**编译期**才能放入常量池，运行期也可以放入常量池，String的**intern**方法就是利用了这个特点。

    再来分析一下一开始的代码：

    ```
    String str1 = "Hello";
    String str2 = new String("Hello");
    String str3 = new String("Hello").intern();
    System.out.println(str1 == str2); // false
    System.out.println(str1 == str3); // true
    ```

    此时的内存结构应该是这样的：

    ![](http://pbhc9u1ue.bkt.clouddn.com/String2.jpg)

    分析new String("Hello") 这段代码，如果后面没有执行intern()方法，那么str2,str3都指向的是堆空间中的对象，也就是图中绿色的那片区域。但是由于是两片不同的空间，地址不同，所以此时 str1 == str2 为false，而且str2 == str3 也是false。

    但是现在 str3 执行了 new String("Hello").intern()，intern()方法会将常量池中的引用返回给str3(因为这里做了赋值)，因为前面的str1 已经在常量池中创建了一个"Hello"字面量，所有str3 接受到的intern()返回的引用与str1 一致，所以str1 == str3 为 true成立。

    在新建字符串对象的时候，我们一般使用下面两种方法：
    - 一种是直接冒号创建：String str = "Hello";
    - 一种是使用构造器：String str = new String("Hello");
    
    不论是上面两种哪种方法，创建字符串对象时都会先检查常量池中是否有该字面量，没有的话就会放入常量池。那么这样的话，intern()是不是就没有用了呢？

    #### intern() 方法的使用

    在前面说 String 对 '+' 重载时说到，String 在使用 '+' 进行字符串拼接时，实质是创建了一个 StringBuilder 对象再调用toString() 方法，但是如果拼接了两个字符串变量，这种拼接之后产生的新的字符串并不在常量池中。
    ```
    String s1 = "Hello";
    String s2 = "World";
    String s3 = s1 + s2;
    String s4 = "Hello" + "World";
    ```

    进行反编译之后
    ```
    String s1 = "Hello";
    String s2 = "World";
    String s3 = new StringBuilder().append("Hello").append("World").toString();
    String s4 = "HelloWorld";
    ```

    究其原因，是因为常量池要保存的是已确定的字面量值。也就是说，对于字符串的拼接，纯字面量和字面量的拼接，会把拼接结果作为常量保存到字符串池。

    如果在字符串拼接中，有一个参数是非字面量，而是一个变量的话，整个拼接操作会被编译成StringBuilder.append，这种情况编译器是无法知道其确定值的。只有在运行期才能确定。

    所以只有运行期才能确定的字符串，就可以使用intern()方法放入常量池，减少字符串的重复创建。


    前面提到了new String("Hello")，也会把Hello放入常量池中，那new String("Hello").intern() 是不是就多余了呢。其实不然，intern() 方法有两个作用，一个是讲字符串放入常量池，另一个是**将常量引用返回**。

    也就是这个值是有返回值的，返回的就是常量的引用。如果将下面的代码

    ```
    String str1 = "Hello";
    String str2 = new String("Hello");
    String str3 = new String("Hello").intern();
    System.out.println(str1 == str2); // false
    System.out.println(str1 == str3); // true
    ```

    修改为：

    ```
        String str1 = "Hello";
        String str2 = new String("Hello");
        String str3 = new String("Hello");
        // 使用新的变量接受返回的引用
        String str4 = str3.intern();
        System.out.println(str1 == str2); // false
        System.out.println(str1 == str3); // false（这里true 不再成立）
        System.out.println(str1 == str4); // true
    ```

    不过这种写法的确没有什么意义，但是对于理解intern()方法和常量池是很有帮助的。