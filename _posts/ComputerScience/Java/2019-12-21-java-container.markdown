---
layout: post
title:  "【Java】Java 总结 02 Java容器"
crawlertitle: "Java Containers"
summary: "Java Containers"
date:   2019-12-21 09:00:00 +0800
categories: posts
tags: 'CS'
author: xusc
bg: "CS.jpg"
---

Java 的容器框架和底层实现，

- `Collection`：Collection 继承了 Iterable 接口，其中 iterator() 方法能够产生一个 Iterator 对象，通过这个对象就可以迭代遍历 Collection 中的元素。属于*迭代器模式*。
  - `Set`：无序、数据不重复
    - `TreeSet`：基于红黑树实现。有序。
    - `HashSet`：基于哈希表实现。无序。读取快，查找的时间复杂度为 O(1)。
      - `LinkedHashSet`：按照元素添加顺序保存，具有 HashSet 的查找效率。
  - `List`：有序、数据可重复
    - [`ArrayList`](#arraylist)：基于动态数组实现，支持随机访问。异步，非线程安全。
    - [`LinkedList`](#linkedlist)：基于双向链表实现，只能顺序访问。异步，非线程安全。
    - `Vector`：和 ArrayList 类似。同步，线程安全。
      - `Stack`：Vector 的子类。
    - `CopyOnWriteArrayList`：读写分离，写操作在一个复制的数组上进行，读操作还是在原始数组中进行。
  - `Queue`
    - `LinkedList`：可以用它来实现双向队列。
    - `PriorityQueue`：基于堆结构实现，可以用它来实现优先队列。
- `Map`
  - `TreeMap`：基于红黑树实现。有序。
  - [`HashMap`](#hashmap)：基于哈希表实现。
    - `LinkedHashMap`：继承自 HashMap。使用双向链表来维护元素的顺序，顺序为插入顺序或者最近最少使用（LRU）顺序。
  - `ConcurrentHashMap`：和 HashMap 实现上类似。线程安全。采用了分段锁（Segment），每个分段锁维护着几个桶（HashEntry），多个线程可以同时访问不同分段锁上的桶，从而使其并发度更高。
  - `HashTable`：和 HashMap 类似，但它是线程安全的。它是遗留类，已弃用，而是使用 ConcurrentHashMap 来支持线程安全。
  - `IdentityHashMap`：使用 `==` 替代 `equals()` 对键进行比较
  - `WeakHashMap`：主要用来实现缓存，通过使用 WeakHashMap 来引用缓存对象，由 JVM 对这部分缓存进行回收。

`Collections` 工具类：提供常用的集合操作方法，和集合的同步实现。

![](#/assets/images/2019/JavaContainer.png)

#### ArrayList
类描述
+ 继承了 AbstractList 抽象类，实现了 List 接口。它是一个数组队列，提供了相关的添加、删除、修改、遍历等功能。
+ 实现了 RandmoAccess 接口。即提供了随机访问功能。
+ 实现了 Cloneable 接口。即覆盖了函数 clone()，能被克隆。
+ 实现 java.io.Serializable 接口。这意味着 ArrayList 支持序列化，*只序列化数组中有元素填充那部分内容*。

构造器
+ `ArrayList()` 构造一个空列表
  + 初始化容量为 0，等到第一次 add 的时候再初始化为*默认大小 **10***。
+ `ArrayList(Collection<?extend E> c)` 构造一个包含指定元素的列表。
+ `ArrayList(int initialCapcity)` 构造一个具有初始容量值得空列表。

操作
- 添加元素时使用 ensureCapacityInternal() 方法来保证容量足够，如果不够时，需要使用 grow() 方法进行扩容，新容量的大小为旧容量的 **1.5** 倍。扩容操作需要调用 Arrays.copyOf() 把原数组整个复制到新数组中，这个操作代价很高，因此最好在创建 ArrayList 对象时就指定大概的容量大小，**减少扩容**操作的次数。
- 删除元素需要调用 System.arraycopy() 将 index + 1 后面的元素都复制到 index 位置上。
- 迭代器两种，一种是指提供向后遍历的 Iterator，另一种是List的专有迭代器 ListIterator。

操作|数组|ArrayList
:-:|:-:|:-:
创建|`int[] a = new int[10]`|`List<Integer> list = new ArrayList<Integer>()`
大小|`a.length`|`list.size()`
排序|`Arrays.sort(a)`|`Collections.sort(list)`
添删改|相当复杂|相应函数
访问元素|`a[0]`|`list.get(0)`

#### LinkedList
类描述
+ 继承了 AbstractSequentialList 抽象类，实现了 List 接口。
+ 实现了 Cloneable 接口。它支持克隆
+ 实现了 Deque 接口。实现了 Deque 所有的可选的操作。
+ 实现了 Serializable 接口。表明它支持序列化。

构造器
+ `LinkedList()` 空构造器
+ `LinkedList(Collection<? extends E> c)` 构造一个包含指定元素的链表

操作
- 元素
  - 基于双向链表实现，Node 类是 LinkedList 中的私有内部类，使用 Node 存储链表节点信息，每个链表存储了 first 和 last 指针。
  - `Node<E> first, last`：LinkedList 的头尾节点，不可序列化。
- 内部实现了 6 种主要的辅助方法，方法的实现都是依靠这 6 种辅助方法实现的。
- 迭代器两种，一种是指提供向后遍历的 Iterator，另一种是 List 的专有迭代器 ListIterator。
- 操作分两类，一类是实现列表接口的方法，一类是实现双端队列接口的方法。
- 推荐使用顺序访问，也就是使用我们的迭代器。

#### HashMap
类描述
+ 继承了 AbstractMap 抽象类，实现了 Map 接口。
+ 实现了 Cloneable 接口。它支持克隆。
+ 实现了 Serializable 接口。表明它支持序列化。
  + table 变量被 transient 所修饰，因此 HashMap 并没有使用默认的序列化机制，而是通过实现 readObject/writeObject 两个方法自定义了序列化的内容。

存储结构：内部包含了一个 Node 类型的数组，继承自 Map 接口的 Entry。Entry 存储着键值对。它包含了四个字段，从 next 字段我们可以看出 Entry 是一个链表。

构造方法
+ HashMap() 
+ HashMap(int initialCapacity) 构造一个具有初始容量值得空哈希表。
+ HashMap(int initialCapacity, float loadFactor) 构造一个具有初始容量值和指定负载因子的空哈希表。
+ HashMap(Map<? extends K, ? extends V> m) 构造一个包含指定元素的哈希表。

操作
+ 插入
  + 使用拉链法来解决冲突。
  + 使用链表的头插法，也就是新的键值对插在链表的头部。
  + HashMap 使用第 0 个桶存放键为 null 的键值对。
  + 一个桶存储的链表长度大于等于 8 时会将链表转换为红黑树。
+ 扩容
  + 相关参数
    + capacity：table 的容量大小，默认为 16。需要注意的是 capacity 必须保证为 2 的 n 次方。
    + size：键值对数量。
    + threshold：size 的临界值，默认为 0.75f。
    + loadFactor：负载因子，table 能够使用的比例，threshold = (int)(capacity * loadFactor)。
  + 当需要扩容时，原来的两倍。
  + 扩容操作同样需要把 oldTable 的所有键值对重新插入 newTable 中，这一步很费时。
+ 计算哈希值 —— 无符号右移 16 位后做异或运算：将高低位二进制特征混合起来

#### 线程安全的容器

##### Vector
- 它的实现与 ArrayList 类似，但是使用了 synchronized 进行同步。
- Vector 比 ArrayList 多一个可以传入 capacityIncrement 参数的构造函数，它的作用是在扩容时使容量 capacity 增长 capacityIncrement。默认情况下 Vector 每次扩容时容量都会**翻倍**。

比较|ArrayList|Vector
:-:|:-:|:-:
是否线程安全|否|是<br/>使用 synchronized 同步**方法**
无参构造初始化容量|0|10
扩容|1.5 倍|默认 2 倍<br/>可设置

SynchronizedList 是 java.util.Collections 中的一个静态内部类，使用 synchronized 同步**代码块**。在多线程的场景中可以直接使用 Vector 类；也可以使用 Collections.synchronizedList(List list) 方法来返回一个线程安全的 List，但在遍历的时候要手动进行同步。

##### CopyOnWriteArrayList
- 写时复制：对集合元素进行写操作时，首先复制一个副本；
- 适用于读多写少的情况。


##### HashTable与ConcurrentHashMap
HashTable
- 数组 + 链表实现，无论key还是value都不能为null，线程安全，实现线程安全的方式是在修改数据时锁住整个 HashTable，效率低。
- 已弃用。

ConcurrentHashMap
- 使用的 CAS 编程方式控制线程安全， 必要时也会使用 synchronized 代码块保证线程安全。
- 读不加锁，写加锁。
