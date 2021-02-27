# String 类能不能被继承？为什么？

> 不能，因为String类有final修饰符，而final修饰的类是不能被继承的，实现细节不允许改变。

## 关于final修饰符

> final从字面理解含义为”最终的、最后的“。在java也同样表示此种含义。
>
> final可以用来修饰变量（包括类属性、对象属性、局部变量和形参）、方法（包括类方法和对象方法）和类。

### 1. final修饰类

> final修饰类即表示此类已经是“最后的、最终的”含义。因此，用final修饰的类不能被继承，即不能拥有自己的子类。
>
> 如果试图对一个已经用final修饰的类进行继承，在编译期间会发生错误。

### 2. final修饰方法

> final修饰的方法表示此方法已经是“最后的、最终的”含义，即此方法不能被重写（可以重载多个final修饰的方法）。
>
> 此处需要注意的一点是：因为重写的前提是子类可以从父类中继承此方法，如果父类中final修饰的方法同时访问控制权限为private，
>
> 将会导致子类中不能直接继承到此方法，因此，此时可以在子类中定义相同的方法名和参数，此时不再产生重写与final的矛盾，而是
>
> 在子类中重新定义了新的方法。

### 3. final 修饰变量

> final修饰的变量表示此变量是“最后的、最终的”含义。一旦定义了final变量并在首次为其显示初始化后，final修饰的变量值不可被改变。
>
> 这里需要注意：
>
> 1. final修饰的类属性和变量属性必须要进行显示初始化赋值。
> 2. 无论对于基本数据类型还是引用数据类型，final修饰的变量都是首次显示初始化后值都不能修改。

### 4. final修饰变量后导致的“宏替换”/"宏变量"问题

> Java 中宏变量/宏替换指的是在java代码中在编译期某些变量能够直接被其本身的值所替换，编译到.class文件中。因此，编译后的.class文件中已经不存在此变量。

#### 不加final

```
// 编译前
public class test {
    public static void main(String[] args) {
        String a = "hello";
        String b = "world";
        String c = a + b;
        System.out.println(c);
    }
}
// 编译后
public class test {
    public test() {
    }
    public static void main(String[] args) {
        String a = "hello";
        String b = "world";
        String c = a + b;
        System.out.println(c);
    }
}
// 里面依然存的是对象的引用
```

#### 加final

````
// 编译前
public class test {
    public static void main(String[] args) {
        final String a = "hello";
        final String b = "world";
        String c = a + b;
        System.out.println(c);
    }
}
// 编译后
public class test {
    public test() {
    }
    public static void main(String[] args) {
        String a = "hello";
        String b = "world";
        String c = "helloworld";
        System.out.println(c);
    }
}
// 直接存的是值本身
````

