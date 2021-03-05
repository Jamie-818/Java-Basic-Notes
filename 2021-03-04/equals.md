# == 和 equals() 的区别？

## 先说结论

> == 在对比基本类型时是**值相等**，在对比引用类型时对比**地址相等**。
>
> equals只能做于引用类型对比，直接对比**值相等**。 **如果对象多个字段时，但是有些字段不想用做对比，可以重写equals**

## 代码实例

### new 对象 "==" 比较

```
/**
 * newString 用 == 对比的是地址，因为都是new的对象，所以应用的地址不一样，导致两个对象不相等，对比是false
 */
private static void newString() {
    String a = new String("123");
    String b = new String("123");
    System.out.println("newString == result: " + (a == b));
}
```

### String常量对象 "==" 比较

```
/**
 * String = xxx 创建的对象，地址指向的是常量池，只有相同的常量，地址唯一，所以两个对象相等，对比结果为：true
 */
private static void stringDemo() {
    String a = "123";
    String b = "123";
    System.out.println("String == result: " + (a == b));
}

```

### new 对象 "equals"比较

```
/**
 * equals对比的是值对比，两个new String 的对象的值是相同的，所以用equals对比结果为：true
 */
private static void stringEquals() {
    String a = new String("123");
    String b = new String("123");
    System.out.println("String equals result: " + (a.equals(b)));
}
```

### 全部代码

```
public class EqualsExample {

    public static void main(String[] args) {
        // newString “==” 对比 false
        newString();
        // string "==" 对比 true
        stringDemo();
        // string "equals" 对比 true
        stringEquals();
    }

    /**
     * newString 用 == 对比的是地址，因为都是new的对象，所以应用的地址不一样，导致两个对象不相等，对比结果为：false
     */
    private static void newString() {
        String a = new String("123");
        String b = new String("123");
        System.out.println("newString == result: " + (a == b));
    }

    /**
     * String = xxx 创建的对象，地址指向的是常量池，只有相同的常量，地址唯一，所以两个对象相等，对比结果为：true
     */
    private static void stringDemo() {
        String a = "123";
        String b = "123";
        System.out.println("String == result: " + (a == b));
    }

    /**
     * equals对比的是值对比，两个new String 的对象的值是相同的，所以用equals对比结果为：true
     */
    private static void stringEquals() {
        String a = new String("123");
        String b = new String("123");
        System.out.println("String equals result: " + (a.equals(b)));
    }

}

```

