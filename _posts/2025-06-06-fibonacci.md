---
layout: post
title:  "斐波那契数列的几种实现"
description: 斐波那契数列的几种实现
tag: [斐波那契数列,递归,java ,zf520,java思录]
date:   2025-06-06 08:50:35 +0800
categories: java
author: ZF520
typora-root-url: ./..
typora-copy-images-to: ./..\assets\images
---

# 斐波那契数列的几种实现

最近有人问笔者，斐波那契数列的实现都有哪些实现方式？

## 递归实现

嗯。最基本最简单实现就是使用递归的方式来处理。

```java
public class Fibonacci {
    public static int fibRecursive(int n) {
        if (n <= 1) {
            return n;
        }
        return fibRecursive(n - 1) + fibRecursive(n - 2);
    }
}
```

这个是最常见的一个解法，但一般这个算法主要用来进行教学使用，而且，在n不是那么大的场景下才合适。比如，当笔者把这个程序在java中实现并允许的时候，如果输入的是Integer.MAX_VALUE，则输出一个异常`StackOverflowError`，这是因为每次循环，都要在栈空间中存储一个值，那么多的递归调用，就会出现栈溢出的问题。

![image-20250606092320367](/assets/images/image-20250606092320367.png)

不过，如果面试的时候遇到了这个问题，那么，一定要使用这个算法来实现，因为，这个实现虽然简单，但却是最容易实现，也最容易给面试官进行讲解的。

在说下，这个算法的时间复杂度和空间复杂度是： 时间复杂度是$O(2^n)$，空间复杂度是$O(n)$。

## 迭代方法

所有使用递归实现的都可以使用循环来实现，所以，迭代方法就是一个for循环方式的实现

```java
public class Fibonacci {
    public static int fibIterative(int n) {
        if (n <= 1) {
            return n;
        }
        int a = 0, b = 1;
        for (int i = 2; i <= n; i++) {
            int temp = b;
            b = a + b;
            a = temp;
        }
        return b;
    }
}
```

算法实现还是没有什么难度，读这段代码，也基本上没啥难度，就是使用两个变量临时存储下中间结果，依次循环下去，也可以得到结果。

算法的复杂度：时间复杂度$O(n)$ , 空间复杂度$O(1)$。 这个算法适用于大部分的场景，只要注意int类型的最大长度就可以了，在具体项目中，可以使用当前算法进行处理。而且，时间复杂度和空间复杂度也都非常小。

## 记忆递归

那刚才的递归实现，有没有可以优化的地方？又是哪些地方是可以节省的？看递归的算法，发现，我们每次递归的时候，都会某个数计算两次，在`n-1`中计算一次，在`n-2`中又计算了一次，这个地方是不是可以优化下？嗯。我们来看记忆递归的实现。

```java
import java.util.HashMap;
import java.util.Map;

public class Fibonacci {
    private static Map<Integer, Integer> memo = new HashMap<>();
    
    public static int fibMemoization(int n) {
        if (n <= 1) {
            return n;
        }
        if (memo.containsKey(n)) {
            return memo.get(n);
        }
        int result = fibMemoization(n - 1) + fibMemoization(n - 2);
        memo.put(n, result);
        return result;
    }
}
```

这个算法在于记忆了每次的计算结果，减少了一次递归的深度。当数据越大的时候，这个优化就越有优势。

未优化前，每个值都需要递归多次（输出的n）

![image-20250606095424506](/assets/images/image-20250606095424506.png)

优化后，同样对`n`进行输出

![image-20250606095556580](/assets/images/image-20250606095556580.png)

每个值只递归了一次。输出也少了很多。

复杂度分析： 时间复杂度$O(n)$，空间复杂度$O(n)$，这个算法就是要在必须使用递归的情况下进行的实现，对递归的一个优化。

## 矩阵快速幂

斐波那契数列的定义：$ F(n) = \begin{cases}  0, & \text{if } n = 0 \\ 1, & \text{if } n = 1 \\ F(n-1) + F(n-2), & \text{if } n > 1  \end{cases}   $

用矩阵来表示递推的关系：

$  \begin{bmatrix} F(n) \\ F(n-1) \end{bmatrix} = \begin{bmatrix} 1 & 1 \\ 1 & 0 \end{bmatrix} \times \begin{bmatrix} F(n-1) \\ F(n-2) \end{bmatrix} $ 

一步一步的推导到`N`次 

$\begin{bmatrix} F(n) \\ F(n-1) \end{bmatrix} = \begin{bmatrix} 1 & 1 \\ 1 & 0 \end{bmatrix}^{n-1} \times \begin{bmatrix} F(1) \\ F(0) \end{bmatrix}$ 

由于$F(1) =1$ 和$F(0) =0$，简化：

$\begin{bmatrix} F(n) \\ F(n-1) \end{bmatrix} = \begin{bmatrix} 1 & 1 \\ 1 & 0 \end{bmatrix}^{n-1} \times \begin{bmatrix} 1 \\ 0 \end{bmatrix}$

最终简化为$2*2$矩阵乘法运算。

具体实现

```java
public static int fibMatrix(int n) {
        if (n == 0) {
            return 0;
        }
    {% raw %}
        int[][] mat = {{1, 1}, {1, 0}};
    {% endraw %}
        int[][] result = matrixPower(mat, n - 1);
        return result[0][0];
    }
    
    private static int[][] matrixMult(int[][] a, int[][] b) {
        int[][] result = new int[2][2];
        result[0][0] = a[0][0] * b[0][0] + a[0][1] * b[1][0];
        result[0][1] = a[0][0] * b[0][1] + a[0][1] * b[1][1];
        result[1][0] = a[1][0] * b[0][0] + a[1][1] * b[1][0];
        result[1][1] = a[1][0] * b[0][1] + a[1][1] * b[1][1];
        return result;
    }
    
    private static int[][] matrixPower(int[][] mat, int power) {
        {% raw %}
        int[][] result = {{1, 0}, {0, 1}};
        {% endraw %}
        while (power > 0) {
            if (power % 2 == 1) {
                result = matrixMult(result, mat);
            }
            mat = matrixMult(mat, mat);
            power /= 2;
        }
        return result;
    }
```

算法复杂度的分析：

时间复杂度：

若`n`为偶数： $A^n = (A^{n/2})^2$

如`n`为奇数： $ A^n = A \times (A^{(n-1)/2})^2 $

所以，每次迭代，相当于把当前值降一个指数。

故： 时间复杂度是$O(log(n))$ ， 而空间复杂度是$O(1)$

![image-20250606102311620](/assets/images/image-20250606102311620.png)

