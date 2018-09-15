# java集合 - Map学习笔记
参考：

- [【Java】HashMap源码分析（JDK1.8）](https://itimetraveler.github.io/2017/11/25/%E3%80%90Java%E3%80%91HashMap%E6%BA%90%E7%A0%81%E5%88%86%E6%9E%90%EF%BC%88JDK1.8%EF%BC%89/)
- 是

    ##一、概述

    java的集合框架中，Map也是一个非常重要和非常常用的数据结构之一。与List不同的是，Map并没有继承 Collection 接口，而是一个独立的接口。下图是Map的继承体系图：

    ![](http://pbhc9u1ue.bkt.clouddn.com/map继承体系.png)

    实现了Map接口的主要的几个实现类有下面几个：

    1. **HashMap**：无序，key和value都**可以**为null，线程不安全。
    2. **HashTable**：无序，key和value都**不可以**为null，线程安全，Hashtable是用来 synchronized 对方法进行加锁，保证了线程安全，但是同时消耗了性能，所以不建议使用。需要线程安全的Hashmap，可以使用 java.util.concurrent 下的 ConcurrentHashMap 。
    3. **LinkedHashMap**：有序，这里的有序指的是**插入的顺序**。LinkedHashMap 在使用 iterator 进行遍历访问时，会按照插入的顺序就行访问。
    4. **TreeMap**：有序，这里的有序指的是**根据键排序**，默认按照键值的升序顺序，也可以指定排序的比较器。在使用TreeMap时，key必须实现Comparable接口或者在构造TreeMap传入自定义的Comparator，否则会在运行时抛出java.lang.ClassCastException类型的异常。


    ## 二、HashMap

    HashMap 是我们平时最常用的Map，想要了解HashMap，那就先来看看HashMap的源码。
    
    ### 1. HashMap的结构

    在JDK1.8之前，HashMap 采用的是位桶(数组) + 链表实现，在发生链表冲突时，会在形成一个链表。当时当链表中的元素和很多时，map的查询效率下降。
    在JDK1.8之后，HashMap 采用了位桶 + 链表 + 红黑树实现。当链表长度长于8时，会讲链表结构自动转换为红黑树。
    
    ![HashMap的结构](http://pbhc9u1ue.bkt.clouddn.com/map结构.png)

    当put一个元素时，首先会计算key的hash值，根据hash值放入桶中对应的位置，也就是图中Entry数组的位置。当数组中已经有了一个元素（发生hash碰撞）时，会在后面生成一个链表，JDK1.8之后，这个链表长度大于8时，会转换为红黑树。

    ### 2. HashMap的扩容操作

    HashMap 中有两个很重要的参数：
    - **initialCapacity 初始容量**：代表了 HashMap 中位桶（数组）的初始长度，不过需要注意的是，HashMap 位桶的长度虽然取决于 initialCapacity 但是每次都会通过tableSizeFor(initialCapacity) 方法来保证为 2 的幂次。 
    
        > 保证桶的长度为2的幂次是因为，在计算key的 hash 时，会对 key 的 hashCode 进行取模，也就是 h % length ,当 length 是2的幂次时，取模就等同于 h & (length -1)，位运算效率要比取模高。更多可以参考:[【Java】HashMap源码分析（JDK1.8）](https://itimetraveler.github.io/2017/11/25/%E3%80%90Java%E3%80%91HashMap%E6%BA%90%E7%A0%81%E5%88%86%E6%9E%90%EF%BC%88JDK1.8%EF%BC%89/)这篇文章。
    
    - **loadFactory 负载因子**：代表位桶在其容量自动扩容前可以达到的当前容量的最大百分比。默认值为0.75。当位桶中的Entry的数量的百分比超过了负载因子，就会出发扩容操作。对于链表来说，查找一个元素的平均时间是O(1+a)，所以负载因子越大，对桶的利用率就越充分，但是会导致查询效率的降低，所以一般采用系统默认的0.75。

    
