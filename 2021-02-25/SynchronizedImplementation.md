# synchronized 关键字底层是如何实现的？它与 Lock 相比优缺点分别是什么？

## synchronized 底层实现

> synchronized 要分为修饰方法及修饰语句块

### synchronized修饰语句块

由一对 monitorenter/monitorexit 指令实现的，monitor 对象是同步的基本实现单元。

```
public class SynchronizedDemo {

    public void method() {
    // synchronized 修饰语句块
        synchronized(this){
            System.out.println("synchronized 代码块");
        }
    }
}
```

```
PS E:\com\jamie\threadpool> javap -c  SynchronizedDemo.class
Compiled from "SynchronizedDemo.java"
public class com.jamie.threadpool.SynchronizedDemo {
  public com.jamie.threadpool.SynchronizedDemo();
    Code:
       0: aload_0
       1: invokespecial #1                  // Method java/lang/Object."<init>":()V
       4: return

  public void method();
    Code:
       0: aload_0
       1: dup
       2: astore_1
       3: monitorenter
       4: getstatic     #2                  // Field java/lang/System.out:Ljava/io/PrintStream;
       7: ldc           #3                  // String synchronized 代码块
       9: invokevirtual #4                  // Method java/io/PrintStream.println:(Ljava/lang/String;)V
      12: aload_1
      13: monitorexit
      14: goto          22
      17: astore_2
      18: aload_1
      19: monitorexit
      20: aload_2
      21: athrow
      22: return
    Exception table:
       from    to  target type
           4    14    17   any
          17    20    17   any
}
```

可以看到第3和13行分别有`monitorenter`和`monitorexit`，**其中 monitorenter 指令指向同步代码块的开始位置，monitorexit 指令则指明同步代码块的结束位置。** 当执行 monitorenter 指令时，线程试图获取锁也就是获取 monitor(monitor对象存在于每个Java对象的对象头中，synchronized 锁便是通过这种方式获取锁的，也是为什么Java中任意对象可以作为锁的原因) 的持有权.当计数器为0则可以成功获取，获取后将锁计数器设为1也就是加1。相应的在执行 monitorexit 指令后，将锁计数器设为0，表明锁被释放。如果获取对象锁失败，那当前线程就要阻塞等待，直到锁被另外一个线程释放为止。

### synchronized修饰方法

synchronized 修饰的方法并没有 monitorenter 指令和 monitorexit 指令，取得代之的是 ACC_SYNCHRONIZED 标识，该标识指明了该方法是一个同步方法，JVM 通过该 ACC*SYNCHRONIZED 访问标志来辨别一个方法是否声明为同步方法，从而执行相应的同步调用。

```
public class SynchronizedDemo2 {
    //  synchronized 修饰方法
    public synchronized void method() {
        System.out.println("synchronized 方法");
    }
}
```

```
PS E:\com\jamie\threadpool> javap -c -s -v  SynchronizedDemo2.class
Classfile /E:/jamie-product/personal/jamie-utils/jamie-thread/target/classes/com/jamie/threadpool/SynchronizedDemo2.class
  Last modified 2021-2-25; size 552 bytes
  MD5 checksum f021f8f92cacc9ead950e1ab5e0f844d
  Compiled from "SynchronizedDemo2.java"
public class com.jamie.threadpool.SynchronizedDemo2
  minor version: 0
  major version: 52
  flags: ACC_PUBLIC, ACC_SUPER
Constant pool:
   #1 = Methodref          #6.#17         // java/lang/Object."<init>":()V
   #2 = Fieldref           #18.#19        // java/lang/System.out:Ljava/io/PrintStream;
   #3 = String             #20            // synchronized 方法
   #4 = Methodref          #21.#22        // java/io/PrintStream.println:(Ljava/lang/String;)V
   #5 = Class              #23            // com/jamie/threadpool/SynchronizedDemo2
   #6 = Class              #24            // java/lang/Object
   #7 = Utf8               <init>
   #8 = Utf8               ()V
   #9 = Utf8               Code
  #10 = Utf8               LineNumberTable
  #11 = Utf8               LocalVariableTable
  #12 = Utf8               this
  #13 = Utf8               Lcom/jamie/threadpool/SynchronizedDemo2;
  #14 = Utf8               method
  #15 = Utf8               SourceFile
  #16 = Utf8               SynchronizedDemo2.java
  #17 = NameAndType        #7:#8          // "<init>":()V
  #18 = Class              #25            // java/lang/System
  #19 = NameAndType        #26:#27        // out:Ljava/io/PrintStream;
  #20 = Utf8               synchronized 方法
  #21 = Class              #28            // java/io/PrintStream
  #22 = NameAndType        #29:#30        // println:(Ljava/lang/String;)V
  #23 = Utf8               com/jamie/threadpool/SynchronizedDemo2
  #24 = Utf8               java/lang/Object
  #25 = Utf8               java/lang/System
  #26 = Utf8               out
  #27 = Utf8               Ljava/io/PrintStream;
  #28 = Utf8               java/io/PrintStream
  #29 = Utf8               println
  #30 = Utf8               (Ljava/lang/String;)V
{
  public com.jamie.threadpool.SynchronizedDemo2();
    descriptor: ()V
    flags: ACC_PUBLIC
    Code:
      stack=1, locals=1, args_size=1
         0: aload_0
         1: invokespecial #1                  // Method java/lang/Object."<init>":()V
         4: return
      LineNumberTable:
        line 3: 0
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
            0       5     0  this   Lcom/jamie/threadpool/SynchronizedDemo2;

  public synchronized void method();
    descriptor: ()V
    flags: ACC_PUBLIC, ACC_SYNCHRONIZED
    Code:
      stack=2, locals=1, args_size=1
         0: getstatic     #2                  // Field java/lang/System.out:Ljava/io/PrintStream;
         3: ldc           #3                  // String synchronized 方法
         5: invokevirtual #4                  // Method java/io/PrintStream.println:(Ljava/lang/String;)V
         8: return
      LineNumberTable:
        line 6: 0
        line 7: 8
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
            0       9     0  this   Lcom/jamie/threadpool/SynchronizedDemo2;
}
SourceFile: "SynchronizedDemo2.java"
```

### 总结

- 两者的区别：同步方式是通过方法中的access_flags中设置ACC_SYNCHRONIZED标志来实现；同步代码块是通过monitorenter和monitorexit来实现

- 我们知道了每个对象都与一个monitor相关联。而monitor可以被线程拥有或释放。

### 扩展

- synchronized 是在 Java 6 之前，monitor 的实现完全是依靠操作系统内部的互斥锁，因为需要进行用户态到内核态的切换，所以同步操作是一个无差别的重量级操作，性能也很低。

- 但在 Java 6 的时候，Java 虚拟机 对此进行了大刀阔斧地改进，提供了三种不同的 monitor 实现，也就是常说的三种不同的锁：偏向锁（Biased Locking）、轻量级锁和重量级锁，大大改进了其性能。

- 同步方法和同步代码块底层都是通过monitor来实现同步的。

## 与 Lock 相比优缺点对比

### 底层实现

- Synchronized：底层使用指令码方式来控制锁的，映射成字节码指令就是增加来两个指令：monitorenter和monitorexit。当线程执行遇到monitorenter指令时会尝试获取内置锁，如果获取锁则锁计数器+1，如果没有获取锁则阻塞；当遇到monitorexit指令时锁计数器-1，如果计数器为0则释放锁。
- Lock：底层是CAS乐观锁，依赖AbstractQueuedSynchronizer类，把所有的请求线程构成一个CLH队列。而对该队列的操作均通过Lock-Free（CAS）操作。

### 优缺点

- synchronized 可以作用于代码块和方法上，而lock只能写在代码里，lock控制颗粒度更细。
- synchronized 在线程异常的时候会自动释放锁，lock不会自动释放锁，需要捕获异常最后finally代码块中显式的释放锁，如果控制的不好，容易出现系统异常。
- synchronized 等待锁的时候不可以控制，如果需要超时机制，需要整个线程进行超时，lock可以直接控制超时。
- synchronized 无法得知是否获取锁成功，lock可以得知。
- 功能复杂性。synchronized 加锁可重入、不可中断、非公平；Lock 可重入、可判断、可公平和不公平、细分读写锁提高效率

### 总结

- 实现层面不一样。synchronized 是 Java 关键字，JVM层面 实现加锁和释放锁；Lock 是一个接口，在代码层面实现加锁和释放锁
- 用法不一样。synchronize可以用在代码块上，方法上。lock只能写在代码里，不能直接修改方法。
- 是否自动释放锁。synchronized 在线程代码执行完或出现异常时自动释放锁；Lock 不会自动释放锁，需要再 finally {} 代码块显式地中释放锁
- 是否一直等待。synchronized 会导致线程拿不到锁一直等待；Lock 可以设置尝试获取锁或者获取锁失败一定时间超时
- 获取锁成功是否可知。synchronized 无法得知是否获取锁成功；Lock 可以通过 tryLock 获得加锁是否成功
- 功能复杂性。synchronized 加锁可重入、不可中断、非公平；Lock 可重入、可判断、可公平和不公平、细分读写锁提高效率