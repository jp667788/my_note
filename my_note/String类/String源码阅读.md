# String源码阅读
详情参考：

- [Java 7 源码学习系列（一）——String](http://www.hollischuang.com/archives/99) 


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
