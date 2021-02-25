# volatile 关键字解决了什么问题，它的实现原理是什么？

## 前言

我们现在的cpu为了提高运算速度，为了解决CPU运算速度与内存读写速度不匹配的矛盾，普遍使用了CPU缓存（也就是L1、L2、L3存储），而我们的多线程执行的时候，数据存储的是在内存，当其他线程修改了数据时，因为cpu执行时，在L1、L2、L3能找到对应的数据，就不会读取内存数据，直接从CPU缓存里面拿.

但是使用cpu缓存在多核CPU时会出现问题，当另外一个线程修改了数值的时候，因为没有通知到cpu更新缓存的操作，所以会导致出现数据问题。

所以CPU制造商制定了一个规矩：当一个CPU修改缓存中的字节时，服务器中其他CPU会被通知，它们的缓存将视为无效。这也叫 `缓存一致性协议`

volatile 关键字，就是JVM虚拟机规范要求实现的关键字，本质就是为了解决在多线程中，当一个cpu内存发现改变，通知到其他cpu缓存失效。

## 实现原理

当知道了cpu缓存一致性协议之后，其实对于 volatile 实现原理就很简单了，本质上，就是触发了cpu的缓存一致性。其实不一定 volatile 修饰的内存才会触发缓存一致性，JVM系统层次的执行语句在底层执行时，也会触发缓存一致性协议。只是 volatile 修饰的内存在每次数据变化之后，其值都会被强制刷入主存，而其他CPU的缓存由于遵守了缓存一致性协议，也会把这个变量的值从主存加载到自己的缓存中。这就保证了一个volatile在并发编程中，其值在多个缓存中是可见的。

```
/**
 * volatile 演示running修改后，但是默认线程独立，导致t1线程阻塞
 */
public class T01_HelloVolatile {

    /*volatile*/ boolean running = true;

    void m() {
        System.out.println("m start");
        while(running){
            //            System.out.println("hello");
        }
        System.out.println("m end!");
    }

    public static void main(String[] args) throws InterruptedException {
        T01_HelloVolatile t = new T01_HelloVolatile();
        new Thread(t::m, "t1").start();
        Thread.sleep(1000);
        t.running = false;
    }

}
```

```
/**
 * 演示volatile修饰的值，会通知到t1线程更新running的值导致循环结束
 */
public class T01_HelloVolatile {

    volatile boolean running = true;

    void m() {
        System.out.println("m start");
        while(running){
            //            System.out.println("hello");
        }
        System.out.println("m end!");
    }

    public static void main(String[] args) throws InterruptedException {
        T01_HelloVolatile t = new T01_HelloVolatile();
        new Thread(t::m, "t1").start();
        Thread.sleep(1000);
        t.running = false;
    }

}
```

```
/**
 * 演示System触发缓存一致性使t1线程更新running的值导致循环结束
 */
public class T01_HelloVolatile {

    /*volatile*/ boolean running = true;

    void m() {
        System.out.println("m start");
        while(running){
            System.out.println("hello");
        }
        System.out.println("m end!");
    }

    public static void main(String[] args) throws InterruptedException {
        T01_HelloVolatile t = new T01_HelloVolatile();
        new Thread(t::m, "t1").start();
        Thread.sleep(1000);
        t.running = false;
    }

}
```



