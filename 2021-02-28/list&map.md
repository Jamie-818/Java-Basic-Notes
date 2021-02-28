# 集合类中的 List 和 Map 的线程安全版本是什么，如何保证线程安全的？

## List的线程安全版本

1. ### vector

2. ### Collections.synchronizeList

3. ### CopyOrWriteArrayList

### 1. vector

> 使用了同步锁，在所有的方法里面都使用了synchronize关键字，使得方法是同步的。

### 2. Collections.synchronizeList

> 也是使用了同步锁，但是和vector的区别：
>
> 1. vector是在方法体上面加synchronize关键字，而synchronizeList是在执行的方法块上面加synchronize关键字，锁的控制方位比vector小。
> 2. synchronizeList可以指定锁的对象，默认的是this。
> 3. synchronize在listIterator方法里面，没有加锁，如果遍历时需要手动去处理。

### 3. CopyOrWriteArrayList

> CopyOrWriteArrayList使用的是插入时复制并加锁，在插入的时候，首先加上ReentrantLock也就是可重入锁，然后复制一个备份出来，在备份上面加上要插入的值，然后把原来的引用指向备份，然后解锁。
>
> 具体步骤：
>
> 1. ReentrantLock.lock();//  加锁。
> 2. Arrays.copyOf(); //  拷贝一个新的数组,数组长度为原来的+1。
> 3.  newElements[len] = e; //  把插入的对象插入到拷贝数组的末尾。
> 4. setArray(newElements); //  将引用指向新数组  
> 5. ReentrantLock.unlock(); //  解锁、
>
> 缺点：
>
> 1. 耗内存（因为拷贝多了一份）
> 2. 实时性不高（如果是插入的时候进行迭代，会读到旧的数组）
>
> 优点：
>
> 1. 解决了其他list集合的多线程迭代问题
> 2. 数据一致性完整。

## 总结

1. vector因为在方法上面加锁，效率低，所以基本生产是不用的。
2. Collections.synchronizeList虽然控制的比vector更精细，但是依然是用了同步锁，所以性能还不是最优，特别是在读多写少的情况下。适合读少写多的场景。
3. CopyOrWriteArrayList和synchronizeList对比时，写操作是比较慢的，但是读操作是非常快的，适合高速读取，但是实时性要求不高的场景。