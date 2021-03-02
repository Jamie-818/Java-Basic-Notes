## LC-70.爬楼梯

算法地址：https://leetcode-cn.com/problems/climbing-stairs/

```
假设你正在爬楼梯。需要 n 阶你才能到达楼顶。
每次你可以爬 1 或 2 个台阶。你有多少种不同的方法可以爬到楼顶呢？
注意：给定 n 是一个正整数。
示例 1：
输入： 2
输出： 2
解释： 有两种方法可以爬到楼顶。
1.  1 阶 + 1 阶
2.  2 阶
示例 2：
输入： 3
输出： 3
解释： 有三种方法可以爬到楼顶。
1.  1 阶 + 1 阶 + 1 阶
2.  1 阶 + 2 阶
3.  2 阶 + 1 阶
```

>  其实计算公式就是 f(n)=f(n-1)+f(n-2)


### 传统递归法-会超时

```
   /**
     * 传统递归法，会超时
     * @param n 楼梯数
     */
    public int climbStairs(int n) {
        if(n == 0 || n == 1 || n == 2){
            return n;
        }
        return climbStairs0(n - 1) + climbStairs0(n - 2);
    }
```

### 优化递归法，计算值记录

```
    /**
     * 优化递归法，计算值记录
     * @param n 楼梯数
     */
    public int climbStairs(int n) {
        if(n == 0 || n == 1 || n == 2){
            return n;
        }
        Map<Integer, Integer> recordMap = new HashMap<>(n + 1);
        recordMap.put(0, 0);
        recordMap.put(1, 1);
        recordMap.put(2, 2);
        return climbStairs1(recordMap, n - 1) + climbStairs1(recordMap, n - 2);
    }

    private int climbStairs1(Map<Integer, Integer> recordMap, int n) {
        if(recordMap.containsKey(n)){
            return recordMap.get(n);
        }
        return climbStairs1(recordMap, n - 1) + climbStairs1(recordMap, n - 2);
    }
```

### 动态规划，使用数组

```
   /**
     * 动态规划，使用数组
     * @param n 楼梯数
     */
    public int climbStairs(int n) {
        if(n == 0 || n == 1 || n == 2){
            return n;
        }
        int[] arrays = new int[n + 1];
        arrays[0] = 0;
        arrays[1] = 1;
        arrays[2] = 2;
        for(int i = 3; i <= n; i++){
            arrays[i] = arrays[i - 1] + arrays[i - 2];
        }
        return arrays[n];
    }
```

### 交换法，交换刚刚计算过前后的值，达到前两位数相加的效果

```
   /**
     * 交换法，交换刚刚计算过前后的值，达到前两位数相加的效果
     * @param n 楼梯数
     */
    public int climbStairs(int n) {
        if(n == 0 || n == 1 || n == 2){
            return n;
        }
        // 假设a是第一个阶梯,b是第二个阶梯
        int a = 1;
        int b = 2;
        // count 用于记录总数
        int c = 0;
        // 因为前面已经有两个台阶了，所以从3算起
        for(int i = 3; i <= n; i++){
            // c是当前台阶，也就是前两台阶相加
            c = a + b;
            // 然后把前两个台阶各自提前一个台阶
            a = b;
            b = c;
        }
        // 最终计算完，c就是当前台阶的数
        return c;
    }
```

