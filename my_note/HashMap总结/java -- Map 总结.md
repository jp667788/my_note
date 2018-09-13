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
    
