# 1. 时间复杂度
是指执行当前算法所消耗的时间
## 1.1 常见的时间复杂度
大O符号表示法, T(n) = O(f(n))<br>
其中n表示数据规模，O(f(n))表示运行算法所需要的指令数，和f(n)成正比

- O(1): Constant Complexity 常数复杂度<br>
- O(log n): Logarithmic Complexity 对数复杂度
- O(n): Liner Complexity 线性复杂度
- O(n^2): N square Complexity 平方复杂度
- O(n^3): N cube Complexity 立方复杂度
- O(2^n): Exponential Growth Complexity 指数复杂度
- O(n!): Factorial 阶乘

<br>

把数字带入n，看看算多少次，一般就可以得出复杂度
```java

public class BigO{

    public static void test(int n) {
        for (int i = 0; i < n; i++) {
           // do something
        }
    } 
}

```

## 1.2 递归调用的时间复杂度
- O( T * depth ): Recursive Complexity 递归时间复杂度<br>
在每个递归函数中，时间复杂度为T， 只进行一次递归调用，递归深度为depth

### 1.2.1 一次递归调用的复杂度

- 二分查找：

```
/**
 * 最多调用了一次递归而已，所以需要log2n才能递归到底，因此时间复杂度为O(logn)
 */
public boolean binarySearch(int[] array, int l, int r, int target) {
    if ( l > r ) return -1;
    int mid = (l + r) >> 2;
    if ( array[mid] == target ) {
        return true;
    } else if ( array[mid] > target ) {
        return binarySearch(array, l, mid - 1, target);
    }  else {
        rerurn binarySearch(array, mid + 1, r, target);
    }
}

```

- 求和

```
/**
 * 递归深度随着n而线性递增，所以时间复杂度为O(n)
 */
public int sum(int n){
    if ( n == 0 ) return 0;
    return n + sum( n - 1 );
}
```

- 求幂
```
/**
 * 递归深度为log n，因为是球需要除以2多少次才能到底
 */
public double pow( double x, int n ) {
    if ( n == 0 ) return 1.0;
    double t = pow(x, n/2);
    if ( n % 2 == 0) return x*t*t;
    return t*t;
}
```


### 1.2.2 多次递归调用的复杂度

```
/**
 * 当n=3时，调用次数为 1 + 2 + 4 + 8 = 15， 所以时间复杂度为O(2^n)，指数级
 */
public int f(int n) {
    if (n==0) return 1;
    return f(n-1) + f(n-1);
}
```

类似的递归树有归并排序，区别在于
1. 上例中树的深度为n，归并为logn
2. 上例中每次处理的数据规模是一样的， 而归并中每个节点处理的数据规模是逐渐缩小的<br>

所以，归并排序每一层处理的数据量为O(n), 同时有O(logn)层，所以时间复杂度为O(nlogn)

# 2. 空间复杂度
空间复杂度可以理解为除了原始序列大小的内存，在算法过程中用到的额外的存储空间。

一个算法所需的存储空间用f(n)表示。S(n)=O(f(n))，其中n为问题的规模，S(n)表示空间复杂度

程序执行时所需存储空间包括以下两部分：

1. 固定部分，这部分空间的大小与输入/输出的数据的个数多少、数值无关。主要包括指令空间（即代码空间）、数据空间（常量、简单变量）等所占的空间。这部分属于静态空间。

2. 可变空间，这部分空间的主要包括动态分配的空间，以及递归栈所需的空间等。这部分的空间大小与算法有关。


# 3. 贪心算法
### 1. 定义
在作出选择时，每次都要选择对自身最为有利的结果，保证自身利益的最大化

贪心策略适用的前提是：局部最优策略能导致产生全局最优解。

### 2. 存在的问题
- 不能保证求得的最后解是最佳的
- 不能用来求最大值或最小值问题
- 只能求满足某些约束条件的可行解的范围

### 3. 算法流程
1. 建立数学模型来描述问题
2. 把求解的问题分成若干个自问题
3. 对每一个子问题求解，得到子问题的局部最优解
4. 把子问题的局部最优解合成原来的一个解

### 4. 伪代码
从问题的某一初始解出发;
while (能朝给定总目标前进异步) {
    利用可行的决策，求出可行解的一个解元素;
   }
由所有解元素组合成问题的一个可行解;



# 4.排序
## 4.1 时间复杂度为O(n^2)
### 4.1.1 冒泡排序
- 一边比较一边向后两两交换，将最大值 / 最小值冒泡到最后一位；
- 经过优化的写法：使用一个变量记录当前轮次的比较是否发生过交换，如果没有发生交换表示已经有序，不再继续排序；
- 进一步优化的写法：除了使用变量记录当前轮次是否发生交换外，再使用一个变量记录上次发生交换的位置，下一轮排序时到达上次交换的位置就停止比较

#### 两两交换
 ```java

public static void bubbleSort(int[] arr) {
	for (int i = 0; i < arr.length; i++) {
    for (int j = 0; j < arr.length - i - 1; j++) {
      if
    }
  } 
}
 ```