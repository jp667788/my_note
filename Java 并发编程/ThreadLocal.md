# ThreadLocal

ThreadLocal 可以解释成线程的局部变量，ThreadLocal 的变量只有当前自身线程可以访问，别的线程访问不了。
ThreadLocal 提供了一种线程安全的方式，彻底避免发生线程冲突

## 基本使用
- 创建：new ThreadLocal\<T\>()
- 设值：local.set()
- 取值：local.get()
- 初始化：withInitial() 可以通过其他线程初始化值，返回一个 SuppliedThreadLocal


## 实现原理
ThreadLocal 中其实并不会真正保存数据，而是保存在线程 **Thread 中的 ThreadLocalMap 中**。
**象，Value为调用ThreadLocal的set方法设置的值。**

### ThreadLocalMap

ThreadLocalMap 是 Thread 类中的 threadLocals 变量，是 ThreadLocal 静态内部类，但是由 Thread 持有，通过 ThreadLocal 访问。

``` java
/* ThreadLocal values pertaining to this thread. This map is maintained  
 * by the ThreadLocal class. */
 ThreadLocal.ThreadLocalMap threadLocals = null;
``` 

- ThreadLocalMap 内部维护一个 Entry 数组

ThreadLocalMap 中的 每个 Entry 都是弱引用 WeakReference ,当这个变量不被其他对象使用时，可以自动回收这个 ThreadLocal 对象，避免内存泄露。

``` java
static class Entry extends WeakReference<ThreadLocal<?>> {  
 /** The value associated with this ThreadLocal. */  
 Object value;  
 //key就是一个弱引用  
 Entry(ThreadLocal<?> k, Object v) {  
 super(k);  
 value = v;  
 }  
}
```

## 内存泄露问题
ThreadLocalMap 中的 Entry 中只有 key 是[[弱引用]]，但是 value 依然是[[强引用]]。
value 的引用链：
![](https://mynoteimage.oss-cn-beijing.aliyuncs.com/note/2021-09-11-061249.jpg)

只有当 Thread 被回收时，value 才有被回收的机会，否则，value 总是会存在一盒强引用。大部分线程会一直存在系统的整个生命周期，这样就会造成 value 对象出现泄露的可能。

![](https://mynoteimage.oss-cn-beijing.aliyuncs.com/note/2021-09-11-064030.jpg)

当 ThreadLocal 不被其他对象引用时，由于 ThreadLocalMap.Entry 中的 key 是弱引用，
会被垃圾回收器回收，（如果是强引用则不会）。
但是此时 ThreadLocalMap.Entry 中的 value 依然是强引用，所以会出现 key 是 null，value 不为 null的情况、

解决办法：在 ThreadLocalMap 进行 set()、get()、remove() 的时候，进行清理，清理 key 为 null 的 value的值。

ThreadLocal 通过 expungeStaleEntry 方法，在 set()、get()、remove() 时，清理value，防止内存泄露。

> **当不需要 ThreadLocal 变量时，主动调用 remove() **

#### 为什么 Entry 的 key 要使用弱引用

## InheritableThreadLocal

使用场景：子线程可以访问父线程的 ThreadLocal 对象。

InheritableThreadLocal 对象重写了：
- T childValue(T parentValue)
- ThreadLocalMap getMap(Thread t)
- void createMap(Thread t, T firstValue)

在创建新的线程时，在创建线程调用 init() 方法时，会将父线程 threadLocalMap 中的元素复制到子线程的 threadLocalMap 中。

注意：
- 变量传递只有在线程创建时才会发生。如果子线程创建前，父线程中还未初始化 inheritableThreadLocals，那么子线程无法访问到父线程中的数据。
因为 只有调用的 InheritableThreadLocal 的 set() 或 get() 方法，才会初始化 inheritableThreadLocals，才会进入 createInheritedMap 方法。
![](https://mynoteimage.oss-cn-beijing.aliyuncs.com/note/2021-09-11-084840.jpg)
createInheritedMap 方法：

``` java
static ThreadLocalMap createInheritedMap(ThreadLocalMap parentMap) {  
    return new ThreadLocalMap(parentMap);  
}

private ThreadLocalMap(ThreadLocalMap parentMap) {  
    Entry[] parentTable = parentMap.table;  
	 int len = parentTable.length;  
	 setThreshold(len);  
	 table = new Entry[len];  

	 for (int j = 0; j < len; j++) {  
		Entry e = parentTable[j];  
		 if (e != null) {  
				ThreadLocal<Object> key = (ThreadLocal<Object>) e.get();  
				 if (key != null) {
				 // 调用 InheritableThreadLocal 重写的方法，返回对象本身
				 Object value = key.childValue(e.value);  
				 // 此时对象 key 和 value 均和父类是同一个对象
				 Entry c = new Entry(key, value);  
				 int h = key.threadLocalHashCode & (len - 1);  
				 while (table[h] != null)  
					h = nextIndex(h, len);  
				 	table[h] = c;  
					size++;  
				 }  
				}  
			}  
}
```

- 变量的赋值就是从主线程的map复制到子线程，它们的value是同一个对象，如果这个对象本身不是线程安全的，那么就会有线程安全问题

## TreadLocalMap 详解

### 构造方法
``` java
/**
 * Construct a new map initially containing (firstKey, firstValue).
 * ThreadLocalMaps are constructed lazily, so we only create
 * one when we have at least one entry to put in it.
 * ThreadLocalMaps 采用懒加载方式，只有在第一次存储时才被创建（get()也会）
 */
ThreadLocalMap(ThreadLocal<?> firstKey, Object firstValue) {
	// 初始化 Entry 数组大小
    table = new Entry[INITIAL_CAPACITY];
	// 计算索引
    int i = firstKey.threadLocalHashCode & (INITIAL_CAPACITY - 1);
	// 新建第一个 Entry 对象
    table[i] = new Entry(firstKey, firstValue);
    size = 1;
	// 设置扩容阈值
    setThreshold(INITIAL_CAPACITY);
}
```
  
对于 & (INITIAL_CAPACITY - 1) 操作，它是对 INITIAL_CAPACITY 取模的位运算算法，由于是位运算比直接 % 取模运算效率高。

### Entry 
![](https://mynoteimage.oss-cn-beijing.aliyuncs.com/note/2021-09-11-083525.png)

slot 代表table表中的一个位置，有下面三种状态：
- Full slot = Full entry ：表示 table 中某个索引存放了一个 entry ，并且该 entry 的 WeakReference key 不为null ，且指向某个 threadlocalObj
- Stale slot = Stale Entry：表示该 entry 的key 为 null，是需要清理的
- Null slot，表示 table 中的某个位置没有 entry，可以用于新的 entry
- run ： table 中两个连续两个 null slot 之间的序列


### getEntry 方法

``` java
private Entry getEntry(ThreadLocal<?> key) {
    int i = key.threadLocalHashCode & (table.length - 1);
    Entry e = table[i];
    if (e != null && e.get() == key)
        return e;
    else
        return getEntryAfterMiss(key, i, e);
}
```

1. 确定索引位置
2. 命中直接返回
3. 调用 getEntryAfterMiss 方法

	![](https://mynoteimage.oss-cn-beijing.aliyuncs.com/note/2021-09-11-084230.jpg)

### getEntryAfterMiss 方法

``` java
	private Entry getEntryAfterMiss(ThreadLocal<?> key, int i, Entry e) {
		Entry[] tab = table;
		int len = tab.length;

		while (e != null) {
			ThreadLocal<?> k = e.get();
			if (k == key)
				return e;
			if (k == null)
				expungeStaleEntry(i);
			else
				i = nextIndex(i, len);
			e = tab[i];
		}
		return null;
	}
```

1. 如果命中，直接返回
2. 如果 key 为空，清理该位置的元素
3. 以[[线性探测]]的方式继续寻找。

![](https://mynoteimage.oss-cn-beijing.aliyuncs.com/note/2021-09-11-084655.jpg)


### expungeStaleEntry 方法

``` java
	private int expungeStaleEntry(int staleSlot) {
		Entry[] tab = table;
		int len = tab.length;
		// expunge entry at staleSlot
		tab[staleSlot].value = null;
		tab[staleSlot] = null;
		// 大小减1
		size--;

		// Rehash until we encounter null
		Entry e;
		int i;
		for (i = nextIndex(staleSlot, len);
			 (e = tab[i]) != null;
			 i = nextIndex(i, len)) {
			ThreadLocal<?> k = e.get();
			if (k == null) {
				e.value = null;
				tab[i] = null;
				size--;
			} else {
				int h = k.threadLocalHashCode & (len - 1);
				if (h != i) {
					tab[i] = null;

					// Unlike Knuth 6.4 Algorithm R, we must scan until
					// null because multiple entries could have been stale.
					// 如果 entry 原本就在其对应 hashcode 所在的位置，则不做操作，完美！
            		// 如果 entry 所在的位置与其对应 h 不一致，则说明此 entry 在 set 的时候
            		// 就遇到了 hash 冲突，而通过线性探测放到了其他位置，而这个时候因为清除
            		// 过期 entry 可能有 null slot 空出来，所以重新安排其位置
					while (tab[h] != null)
						h = nextIndex(h, len);
					// 直到找到距离 rehash 后最近的空位置
					tab[h] = e;
				}
			}
		}
		return i;
	}
```

1. 显示的将该位置的 Entry.value 和  Entry 引用置空
2. 大小减1
3. 使用线性探测往下继续寻找
	1. 如果 Entry.key 是空，重复步骤1
	2. 否则 rehash Entry.key ，重新调整位置，尽可能靠近 rehash 后的位置

![](https://mynoteimage.oss-cn-beijing.aliyuncs.com/note/2021-09-11-090215.jpg)


### set 方法
``` java
	private void set(ThreadLocal<?> key, Object value) {

		// 这里没有像get 那样一上来就判断直接命中的情况，因为替换原来位置的元素和直接新增新的元素的比例是一样的。

		Entry[] tab = table;
		int len = tab.length;
		int i = key.threadLocalHashCode & (len-1);

		for (Entry e = tab[i];
			 e != null;
			 e = tab[i = nextIndex(i, len)]) {
			ThreadLocal<?> k = e.get();

			if (k == key) {
				e.value = value;
				return;
			}

			if (k == null) {
				replaceStaleEntry(key, value, i);
				return;
			}
		}

		tab[i] = new Entry(key, value);
		int sz = ++size;
		if (!cleanSomeSlots(i, sz) && sz >= threshold)
			rehash();
	}
```

1. 获取 key hash (索引位置)
2. 从该位置循环
	1. 如果 key 相等，命中返回
	2. key 是空，调用 replaceStaleEntry 清除元素
3. 找到hash后第一个空位置
4. 放入新的 Entry
5. 调用 cleanSomeSlots 清理 Slot
6. 判断是否需要进行扩容

![](https://mynoteimage.oss-cn-beijing.aliyuncs.com/note/2021-09-11-091336.jpg)

### cleanSomeSlots 方法
探索性的清理 stale slot，使用时间复杂度是log 级别的扫描。在不扫描元素，但是会残留垃圾和O(n)复杂度的扫描全部元素中间做了平衡。

``` JAVA
	private boolean cleanSomeSlots(int i, int n) {
		boolean removed = false;
		Entry[] tab = table;
		int len = tab.length;
		do {
			i = nextIndex(i, len);
			Entry e = tab[i];
			if (e != null && e.get() == null) {
				n = len;
				removed = true;
				i = expungeStaleEntry(i);
			}
		} while ( (n >>>= 1) != 0);
		return removed;
	}
```

1. 获取索引位置
2. 如果需要清理
	1. removed 标识置为 true 
	2. 调用 expungeStaleEntry 清理
3. 返回 removed

### replaceStaleEntry 方法

### remove 方法
``` java
	private void remove(ThreadLocal<?> key) {
		Entry[] tab = table;
		int len = tab.length;
		int i = key.threadLocalHashCode & (len-1);
		for (Entry e = tab[i];
			 e != null;
			 e = tab[i = nextIndex(i, len)]) {
			if (e.get() == key) 
				// Entry 的 clear 继承自 WeakReference，会使和 ThreadLocal 之间的连接断开，使该 Entry 成为 Stale Slot
				e.clear();
				// 清理 Stale Slot
				expungeStaleEntry(i);
				return;
			}
		}
	}
```

1. 获取该 key 的 hash 位置
2. 线性探测，找到 key 所在的位置
3. 释放与 threadLocal 对象的连接，称为 Stale slot
4. 调用 expungeStaleEntry ，清理 Stale slot


### 用途
- Spring 事务