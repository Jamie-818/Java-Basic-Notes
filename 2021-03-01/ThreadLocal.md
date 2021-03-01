# ThreadLocal 实现原理是什么？

## 1. ThreadLocal作用

- 提供线程内的局部遍历。不同线程之间不会相互干扰，这种变量在线程的生命周期内起作用，减少同一个线程内多个函数或组件之间一些公共变量传递的复杂性。

- 总结

  1. 线程并发：在多线程并发的场景下。

  2. 传递数据：可以通过通过ThreadLocak在同一个线程，不同组件中传递公共变量。
  3. 线程隔离：每个线程的变量都是独立的，不会互相影响。

## 2. 基本使用

### 2.1 常用方法


| 方法声明 | 描述  |
|  ------  | --- ----|
|ThreadLocal()|创建ThreadLocal对象 |
|public void set( T value)|设置当前线程绑定的局部变量 |
|public T get()|获取当前线程绑定的局部变量|
|public void remove()|移除当前线程绑定的局部变量|

### 2.2 使用案例

```
public class MyDemo {

    private static ThreadLocal<String> tl = new ThreadLocal();

    private String content;

    private String getContent() {
        return tl.get();
    }

    private void setContent(String content) {
        tl.set(content);
    }

    public static void main(String[] args) {
        MyDemo demo = new MyDemo();
        for(int i = 0; i < 5; i++){
            Thread thread = new Thread(() -> {
                //  获取当前线程的名字
                demo.setContent(Thread.currentThread().getName() + "的数据");
                System.out.println("-----------------------");
                //  获取当前线程的数据
                System.out.println(Thread.currentThread().getName() + "--- " + demo.getContent());
            });
            thread.setName("线程" + i);
            thread.start();
        }
    }

}
```

打印结果: <img src="https://image-show.oss-cn-shenzhen.aliyuncs.com/typora_img/20210123210812106.png" alt="在这里插入图片描述"> 从结果来看，这样很好的解决了多线程之间数据隔离的问题，十分方便。

## 3. ThreadLocal的核心方法源码

 基于ThreadLocal的内部结构，我们继续分析它的核心方法源码，更深入的了解其操作原理。

除了构造方法之外， ThreadLocal对外暴露的方法有以下4个：

| 方法声明                   | 描述                         |
| -------------------------- | ---------------------------- |
| protected T initialValue() | 返回当前线程局部变量的初始值 |
| public void set( T value)  | 设置当前线程绑定的局部变量   |
| public T get()             | 获取当前线程绑定的局部变量   |
| public void remove()       | 移除当前线程绑定的局部变量   |

 以下是这4个方法的详细源码分析(为了保证思路清晰, ThreadLocalMap部分暂时不展开,下一个知识点详解)

### 3.1 set方法

**（1 ) 源码和对应的中文注释**

```
    /**
     * 设置当前线程对应的ThreadLocal的值
     * @param value 将要保存在当前线程对应的ThreadLocal的值
     */
    public void set(T value) {
        // 获取当前线程对象
        Thread t = Thread.currentThread();
        // 获取此线程对象中维护的ThreadLocalMap对象
        ThreadLocalMap map = getMap(t);
        // 判断map是否存在
        if(map != null){
            // 存在则调用map.set设置此实体entry
            map.set(this, value);
        }else{
            // 1）当前线程Thread 不存在ThreadLocalMap对象
            // 2）则调用createMap进行ThreadLocalMap对象的初始化
            // 3）并将 t(当前线程)和value(t对应的值)作为第一个entry存放至ThreadLocalMap中
            createMap(t, value);
        }
    }

    /**
     * 获取当前线程Thread对应维护的ThreadLocalMap
     * @param t the current thread 当前线程
     * @return the map 对应维护的ThreadLocalMap
     */
    ThreadLocalMap getMap(Thread t) {
        return t.threadLocals;
    }

    /**
     * 创建当前线程Thread对应维护的ThreadLocalMap
     * @param t          当前线程
     * @param firstValue 存放到map中第一个entry的值
     */
    void createMap(Thread t, T firstValue) {
        //这里的this是调用此方法的threadLocal
        t.threadLocals = new ThreadLocalMap(this, firstValue);
    }
```

**（2 ) 代码执行流程**

 A. 首先获取当前线程，并根据当前线程获取一个Map

 B. 如果获取的Map不为空，则将参数设置到Map中（当前ThreadLocal的引用作为key）

 C. 如果Map为空，则给该线程创建 Map，并设置初始值

### 3.2 get方法

**（1 ) 源码和对应的中文注释**

```
/**
     * 返回当前线程中保存ThreadLocal的值
     * 如果当前线程没有此ThreadLocal变量，
     * 则它会通过调用{@link #initialValue} 方法进行初始化值
     * @return 返回当前线程对应此ThreadLocal的值
     */
    public T get() {
        // 获取当前线程对象
        Thread t = Thread.currentThread();
        // 获取此线程对象中维护的ThreadLocalMap对象
        ThreadLocalMap map = getMap(t);
        // 如果此map存在
        if(map != null){
            // 以当前的ThreadLocal 为 key，调用getEntry获取对应的存储实体e
            ThreadLocalMap.Entry e = map.getEntry(this);
            // 对e进行判空 
            if(e != null){
                @SuppressWarnings("unchecked")
                // 获取存储实体 e 对应的 value值
                // 即为我们想要的当前线程对应此ThreadLocal的值
                T result = (T)e.value;
                return result;
            }
        }
        /*
        	初始化 : 有两种情况有执行当前代码
        	第一种情况: map不存在，表示此线程没有维护的ThreadLocalMap对象
        	第二种情况: map存在, 但是没有与当前ThreadLocal关联的entry
         */
        return setInitialValue();
    }

    /**
     * 初始化
     * @return the initial value 初始化后的值
     */
    private T setInitialValue() {
        // 调用initialValue获取初始化的值
        // 此方法可以被子类重写, 如果不重写默认返回null
        T value = initialValue();
        // 获取当前线程对象
        Thread t = Thread.currentThread();
        // 获取此线程对象中维护的ThreadLocalMap对象
        ThreadLocalMap map = getMap(t);
        // 判断map是否存在
        if(map != null)
        // 存在则调用map.set设置此实体entry
        {
            map.set(this, value);
        }else{
            // 1）当前线程Thread 不存在ThreadLocalMap对象
            // 2）则调用createMap进行ThreadLocalMap对象的初始化
            // 3）并将 t(当前线程)和value(t对应的值)作为第一个entry存放至ThreadLocalMap中
            createMap(t, value);
        }
        // 返回设置的值value
        return value;
    }
```

**（2 ) 代码执行流程**

 A. 首先获取当前线程, 根据当前线程获取一个Map

 B. 如果获取的Map不为空，则在Map中以ThreadLocal的引用作为key来在Map中获取对应的Entry e，否则转到D

 C. 如果e不为null，则返回e.value，否则转到D

 D. Map为空或者e为空，则通过initialValue函数获取初始值value，然后用ThreadLocal的引用和value作为firstKey和firstValue创建一个新的Map

总结: **先获取当前线程的 ThreadLocalMap 变量，如果存在则返回值，不存在则创建并返回初始值。**

### 3.3 remove方法

**（1 ) 源码和对应的中文注释**

```
    /**
     * 删除当前线程中保存的ThreadLocal对应的实体entry
     */
    public void remove() {
        // 获取当前线程对象中维护的ThreadLocalMap对象
        ThreadLocalMap m = getMap(Thread.currentThread());
        // 如果此map存在
        if(m != null){
            // 存在则调用map.remove
            // 以当前ThreadLocal为key删除对应的实体entry
            m.remove(this);
        }
    }
```

**（2 ) 代码执行流程**

 A. 首先获取当前线程，并根据当前线程获取一个Map

 B. 如果获取的Map不为空，则移除当前ThreadLocal对象对应的entry

### 3.4 initialValue方法

```
   /**
     * 返回当前线程对应的ThreadLocal的初始值
     *
     * 此方法的第一次调用发生在，当线程通过get方法访问此线程的ThreadLocal值时
     * 除非线程先调用了set方法，在这种情况下，initialValue 才不会被这个线程调用。
     * 通常情况下，每个线程最多调用一次这个方法。
     *
     * p 这个方法仅仅简单的返回null {@code null};
     * 如果程序员想ThreadLocal线程局部变量有一个除null以外的初始值，
     * 必须通过子类继承{@code ThreadLocal} 的方式去重写此方法
     * 通常, 可以通过匿名内部类的方式实现
     * @return 当前ThreadLocal的初始值
     */
    protected T initialValue() {
        return null;
    }
```

 此方法的作用是 返回该线程局部变量的初始值。

（1） 这个方法是一个延迟调用方法，从上面的代码我们得知，在set方法还未调用而先调用了get方法时才执行，并且仅执行1次。

（2）这个方法缺省实现直接返回一个`null`。

（3）如果想要一个除null之外的初始值，可以重写此方法。（备注： 该方法是一个`protected`的方法，显然是为了让子类覆盖而设计的）

## 4. ThreadLocal的内部结构

### 4.1 常见的误解

 如果我们不去看源代码的话，可能会猜测`ThreadLocal`是这样子设计的：每个`ThreadLocal`都创建一个`Map`，然后用线程作为`Map`的`key`，要存储的局部变量作为`Map`的`value`，这样就能达到各个线程的局部变量隔离的效果。这是最简单的设计方法，JDK最早期的`ThreadLocal` 确实是这样设计的，但现在早已不是了。 <img src="https://img-blog.csdnimg.cn/20210124115504330.png" alt="在这里插入图片描述">

### 4.2 现在的设计

 但是，JDK后面优化了设计方案，在JDK8中 `ThreadLocal`的设计是：每个`Thread`维护一个`ThreadLocalMap`，这个Map的`key`是`ThreadLocal`实例本身，`value`才是真正要存储的值`Object`。

具体的过程是这样的：

-  每个Thread线程内部都有一个Map (ThreadLocalMap)。
-  Map里面存储ThreadLocal对象（key）和线程的变量副本（value） 。
-  Thread内部的Map是由ThreadLocal维护的，由ThreadLocal负责向map获取和设置线程的变量值。 
-  对于不同的线程，每次获取副本值时，别的线程并不能获取到当前线程的副本值，形成了副本的隔离，互不干扰。 <img src="https://img-blog.csdnimg.cn/20210124120125266.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDA1MDE0NA==,size_16,color_FFFFFF,t_70" alt="在这里插入图片描述"> 

### 4.3 这样设计的好处

 这个设计与我们一开始说的设计刚好相反，这样设计有如下两个优势：

- 这样设计之后每个`Map`存储的`Entry`数量就会变少。因为之前的存储数量由`Thread`的数量决定，现在是由`ThreadLocal`的数量决定。在实际运用当中，往往ThreadLocal的数量要少于Thread的数量。
- 当`Thread`销毁之后，对应的`ThreadLocalMap`也会随之销毁，能减少内存的使用。

## 总结

> ThreadLocal 的实现，实际上是通过当前线程存储了创建的ThreadLocal对象，ThreadLocal存储了ThreadLocalMap,Map里面存储了值实现的，所以每个线程，只能获取到自己创建的ThreadLocal，也就只能获取到当前线程的存储值，达到了内存隔离，及不同组件数据传递不受多线程干扰的目的。