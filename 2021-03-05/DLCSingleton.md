# DLC模式下的单例实现是否线程安全

## 什么是DLC？

DLC 指的是 Double Check Lock 双端检测锁机制

一般常见的懒汉单例模式下的对象是否为空检查

核心代码

```
  if(instance == null){
            synchronized(SingletonDemo.class){
                if(instance == null){
                    instance = new SingletonDemo();
                }
            }
        }
```

当多线程的情况下，防止在第一条线程创建对象之时，另外一条线程同时进入判断，这时候对象因为还没实例化，所以会出现多条线程同时创建对象，instance对象只会生成一个。

完整源码

```
public class SingletonDemo {

    private static SingletonDemo instance = null;

    private SingletonDemo() {
        System.out.println(Thread.currentThread().getName() + " 我是构造方法SingletonDemo()");
    }

    /**
     * DCL模式 Double Check Lock 双端检测锁机制
     */
    private static SingletonDemo getInstance() {
        if(instance == null){
            synchronized(SingletonDemo.class){
                if(instance == null){
                    instance = new SingletonDemo();
                }
            }
        }
        return instance;
    }
}
```



## CPU指令分析

> 首先给出结论：DLC（双端检锁）机制不一定线程安全，原因是有指令重排序的存在。

### 什么叫指令重排? 

计算机在执行程序时，为了提高性能，编译器和处理器的常常会对指令做重排，一般分为以下三种

![image-20210305202217102](https://image-show.oss-cn-shenzhen.aliyuncs.com/typora_img/image-20210305202217102.png)

单线程环境里面确保程序最终执行结果和代码顺序执行的结果一致。

处理器在进行重排序时必须考虑指令之间的**数据依赖性**

多线程环境中线程交替执行，由于编译器优化重排的存在，两个线程中使用的变量是否一致性是无法确认的，结果无法预存。

### DLC线程不安全原因分析

原因在于某一个线程执行到第一次检测，读取到instance不到null时，instance的引用对象**可能没有完成初始化**

instance = new SingletonDemo();  // 可以分为以下3步完成（伪代码）

```
memory = allocate(); // 1.分配对象内存空间
instance(memory);    // 2.初始化对象
instance = memory;   // 3.设置instance指向刚分配的内存地址，此时instance!=null
```

步骤2和步骤3 **不存在数据依赖关系**，而且无论重排前还是重排后程序的执行结果在单线程中并没有改变

因此这种重排优化是允许的。

```
memory = allocate(); // 1.分配对象内存空间
instance = memory;   // 3.设置instance指向刚分配的内存地址，此时instance!=null ,但是对象还没有初始化完成
instance(memory);    // 2.初始化对象
```

但是指令重排只会保证串行语义的执行的一致性(单线程)，并不会关心多线程间的语义一致性。

**所以当一条线程访问instance不为null时，由于instance实例未必已初始化完成，也就造成了线程安全问题**

### 解决方案

在需要单例生成的对象定义时加上`volatile`关键字，`volatile` 关键字，通过添加内存屏障指令，可以禁止CPU的指令重排优化，使DLC模式下创建的对象达到线程安全的效果。

```
private static volatile SingletonDemo instance = null;
```

## 完整代码

```
public class SingletonDemo {

    private static volatile SingletonDemo instance = null;

    private SingletonDemo() {
        System.out.println(Thread.currentThread().getName() + " 我是构造方法SingletonDemo()");
    }

    /**
     * DCL模式 Double Check Lock 双端检测锁机制
     */
    private static SingletonDemo getInstance() {
        if(instance == null){
            synchronized(SingletonDemo.class){
                if(instance == null){
                    instance = new SingletonDemo();
                }
            }
        }
        return instance;
    }

}
```



