# Java 自动拆箱和自动装箱学习笔记
### 详情参考以下
- [深入剖析Java中的装箱和拆箱](https://www.cnblogs.com/dolphin0520/p/3780005.html)
- [JDK自动拆箱下，三目运算符的潜规则](https://blog.csdn.net/tiwerbao/article/details/34244139)
- [Java 自动装箱与拆箱的实现原理](https://www.jianshu.com/p/0ce2279c5691)
- [Autoboxing and Unboxing](https://docs.oracle.com/javase/tutorial/java/data/autoboxing.html)

## 1. 概述
Java 中的自动装箱和自动拆箱算是一种语法糖，也就是在编译阶段编译器在合适的情况下帮我们的做了自动拆箱和自动装箱。

众所周知，Java 中的基本数据类型并不是对象，为了解决在一定切情况下我们需要使用对象的时候，Java 为我们提供了每个基本类型对应的包装类，如下表：

| 基本数据类型 | 数据类型包装类 |
| ----------- | ----------- |
| byte(1字节) | Byte |
| short(2个字节) | Short |
| int(4个字节) | Integer |
| long(8个字节) | Long |
| float(4个字节) | Float |
| double(8个字节) | Double |
| boolean | Boolean |

- 当基本数据类型转换成数据类型包装类时，称之为**装箱**；
- 当数据类型包装类转换成基本数据类型时，称之为**拆箱**。

```
    Integer i = 10; // 装箱
    int num = i; // 拆箱
```

## 2. 自动拆箱和自动装箱的实现原理。
对于上面一段代码进行反编译可以得到：
```
    0: bipush        10
    2: invokestatic  #2                  // Method java/lang/Integer.valueOf:(I)Ljava/lang/Integer;
    5: astore_1
    6: aload_1
    7: invokevirtual #3                  // Method java/lang/Integer.intValue:()I
    10: istore_2
    11: return
```

可以看出：装箱是Integer.valueOf() 方法，而拆箱是调用Integer.intValue() 方法，其他类型亦是如此。**装箱时调用包装类的valueOf()方法，拆箱是调用包装类xxValue()方法。**

## 3. 自动拆箱和自动装箱的发生时机
在下面的两种情况时，会发生**自动装箱**：

- 基本类型作为参数传递到相应的包装类型方法
- 基本类型分配到相应的包装类型变量

对应如下代码：
```
    // 基本类型作为参数传递到相应的包装类型方法
    List<Integer> li = new ArrayList<>();
        for (int i = 1; i < 50; i += 2)
            li.add(i);

    // 基本类型分配到相应的包装类型变量
    Integer i = 10;
```

在下面的两种情况时，会发生**自动拆箱**：

- 包装类型作为参数传递给基本类型的方法
- 包装类型分配到基本类型的变量

> 同时在进行 +，- 时，也会发生自动拆箱，因为 Integer 或其他包装类型对象无法使用运算符。所以编译时，如果包装类型使用运算符进行运算，会先进行自动拆箱再进行运算。
> 
> 还有一种情况，就是在使用**三目运算符**的情况下，可能会发生自动拆箱。当返回的两个参数并不是同一种数据类型时，编译器会进行自动拆箱，向下转型。只要一个运算中有不同的类型，涉及到类型转换，那么编译器会往下（基本类型）转型，再进行运算，如下：
> ```
        Double db1 = null;
        Long l2 = null;
        boolean flag =false;
        Double db2 = (flag) ? db1 : l2;
> ```
> 其中 db1 l2 不是同一种类型的参数，编译器会进行自动拆箱。

对应如下代码：
```
    // 包装类型作为参数传递给基本类型的方法
    Integer i = new Integer(10);
    int i2 = getIntVal(i);
    private static int getIntVal(int i){
        return i;
    }

    // 基本类型分配到相应的包装类型变量
    int i  = new Integer(10);

```

## 4. 关于包装类 valueOf() 方法
首先看下面一段代码：
```
    Integer i1 = 10;
    Integer i2 = 10;
    Integer i3 = 200;
    Integer i4 = 200;
    System.out.println(i1 == i2); // true
    System.out.println(i3 == i4); // false
```

为什么会产生上面的情况，前面提到，自动装箱调用的时valueOf() 方法，所以先看一下valueOf() 的源码：
```
    public static Integer valueOf(int i) {                                          
        //  IntegerCache.low == -128   IntegerCache.high == 127                  
        if (i >= IntegerCache.low && i <= IntegerCache.high)
            return IntegerCache.cache[i + (-IntegerCache.low)];
        return new Integer(i);
    }
```

这段代码的意思时，每次调用Integer.valueOf() 方法时，如果在-128-127之间就会直接从缓存中取。这样做的原因是，我们在使用Integer 时大部分使用的数字都较小，这样可以提高效率，节省空间。

不仅仅是Integer，其他包装类有缓存：

- Boolean 缓存了 Booelan.TRUE 和 Booelan.FALSE
- Short 缓存了 -128 - 127 之间的值
- Long 缓存了 -128 - 127 之间的值
- Byte 数值有限，所以全部都被缓存。
- Character 缓存了'\u0000' 到 '\u007F'

只有double和float的自动装箱代码没有使用缓存，而double、float是浮点型的，没有特别的热的（经常使用到的）数据的，缓存效果没有其它几种类型使用效率高
 
## 5. 自动拆箱和自动装箱的优缺点

- 自动拆箱和自动装箱的引入方便了我们编写程序，提高编程效率

虽然自动拆箱和自动装箱很方便，但是我们在使用是需要注意自动拆装箱所带来的问题：

- 由于包装类型是对象，就可能在我们不经意的情况下产生了空指针异常。如果对一个空的包装类型对象进行拆箱操作，就会抛出空指针异常。
- 自动拆装箱会较为消耗性能，在性能敏感且操作数量大的情况下，性能会明显下降。

所以要**建议避免无意中的装箱、拆箱行为**



