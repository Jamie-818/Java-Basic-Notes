# HashMap 与 ConcurrentHashMap 的实现原理是怎样的？ConcurrentHashMap 是如何保证线程安全的？

## HashMap实现原理

从字面可以知道，这首先是一个Map，也就是<K,V>形式的键值对，其次是底层是哈希实现的。

我们一般说实现原理，不外乎三种，一是底层存储数据结构，二是插入原理，三是获取原理。

### 1. 底层存储数据结构

HashMap底层的存储结构是哈希表，哈希表的特点是结合了数组和链表的特点，

数组用于快速定位K的位置，下标为K的hashCode计算值，value为链表的头节点。在hash中，数组也叫做桶。

链表用于快速查找对应的值。链表才是存储map里面对应的值。

### 2.插入原理-map.put(k,v)实现原理

**第一步**：首先将k,v封装到Node对象当中（节点）。

**第二步**：它的底层会调用K的hashCode()方法得出hash值。

**第三步**：通过哈希表函数/哈希算法，将hash值转换成数组的下标，下标位置上如果没有任何元素，就把Node添加到这个位置上。如果说下标对应的位置上有链表。此时，就会拿着k和链表上每个节点的k进行equal。如果所有的equals方法返回都是false，那么这个新的节点将被添加到链表的末尾。如其中有一个equals返回了true，那么这个节点的value将会被覆盖。

### 3.获取原理-map.get(k)实现原理

**第一步**：先调用k的hashCode()方法得出哈希值，并通过哈希算法转换成数组的下标。

**第二步**：通过上一步哈希算法转换成数组的下标之后，在通过数组下标快速定位到某个位置上。重点理解如果这个位置上什么都没有，则返回null。如果这个位置上有单向链表，那么它就会拿着参数K和单向链表上的每一个节点的K进行equals，如果所有equals方法都返回false，则get方法返回null。如果其中一个节点的K和参数K进行equals返回true，那么此时该节点的value就是我们要找的value了，get方法最终返回这个要找的value。

### 扩展：为何随机增删、查询效率都很高的原因是？

**原因**：增删是在链表上完成的，而查询只需扫描部分，则效率高。

HashMap集合的key，会先后调用两个方法，hashCode and equals方法，这两个方法都需要重写。

## ConcurrentHashMap 实现原理

从字面可以知道，底层结构依然是HashMap，但是加了Concurrent说明它是**线程安全**的。

和HashMap不一样的是数据value和链表都使用了volatile修饰，保证了获取时的可见性。

### 1.插入原理

虽然链表的value使用volatile修饰，但是不能保证并发的原子性，所以插入的时候，仍然需要加锁处理

**第一步**：尝试获取锁，如果获取失败，则肯定有其他线程存在竞争，则利用 `scanAndLockForPut()`  自旋获取锁。

第二步：如果重试的次数达到了  `MAX_SCAN_RETRIES(单核1多核64)`  则改为阻塞锁获取，保证能获取成功。

**第三步**：和HashMap的put流程一致。（jdk1.7存储value的对象为HashEntry，jdk1.8存储的对象为Node，其原理是一致的）。

最后一步：解除锁。

### 2.获取原理

和HashMap一致，但是因为链表里面存储的value值是通过volatile修饰的，保证了内存可见性，所以每次获取都是最新值。

## ConcurrentHashMap 是如何保证线程安全的?

**JDK1.8之前**：采用了分段锁技术，其中 Segment 继承于 ReentrantLock。不会像 HashTable 那样不管是 put 还是 get 操作都需要做同步处理，理论上 ConcurrentHashMap 支持 CurrencyLevel (Segment 数组数量)的线程并发。每当一个线程占用锁访问一个 Segment 时，不会影响到其他的 Segment。

**JDK1.8**：取消segments字段，抛弃了 Segment 的分段锁，采用了CAS + synchronized 来保证并发安全性，直接对每一行数据进行加锁，进一步减少并发冲突的概率。。

