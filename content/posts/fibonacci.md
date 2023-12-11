+++
title = 'Fibonacci'
date = 2021-12-22T23:34:11+08:00
draft = false
tag = '11'
+++

最近参与面试了几个候选人，面试题中有一道计算Fibonacci数列的，发现自己对这道题的掌握也不怎么充分，所以写篇文章整理一下。

Fibonacci数列的解法，复杂度来区分大概有指数型,常数型和对数型几种,下面一一介绍。

### 普通递归
一般最常见，最容易理解的解法就是递归法了，代码如下：
```cpp
int fibo(int n)
{
    if (0 == n) {
        return 0;
    }
    else if (1 == n) {
        return 1;
    }
    else {
        return fibo(n - 1) + fibo(n - 2);
    }
}
```
递归法最大的问题在于它的容易造成栈溢出(尾递归除外), 其次是效率很低, 存在太多的重复计算。把具体的步骤分解为树状，形式如下：
```cpp
                   F(5)
                /        \
            F(4)         F(3)
          /    \        /     \
      F(3)    F(2)     F(2)    F(1)
    /   \     /   \     /  \
  F(2) F(1) F(1) F(0) F(1) F(0)
 /  \
F(1) F(0)

```
这里可以很直观的看到，为了计算F(5), 我们需要计算多次f(3)和f(2),这些都是不必要的。而这种解法的时间复杂度，是O($(\frac{1 + \sqrt5}{2})^n$)，或者也有人粗略的记为$O(2^n)$。

### 迭代
为了对此进行优化，我们可以利用类似的DP的方法，将曾经计算过的结果都保存下来，代码如下：

```cpp
int fibo(int n)
{
    std::vector<int> result(n+1, 0);

    result[1] = 1;

    for (int i = 2; i <= n; ++i)
    {
        result[i] = result[i-1] + result[i-2];
    }

    return result[n];
}

```
由代码可以看出,其实我们只需要记录最近的两个值就可以了, 所以可以得到常见的迭代版本:
```cpp
int fibo(int n)
{
    if (n < 1) {
        return 0;
    }
    else if (n == 1) {
        return 1;
    }
    else {
        int a = 0;
        int b = 1;

        for (int i = 2; i <= n; ++i) {
            int tmp = a + b;
            a = b;
            b = tmp;
        }

        return b;
    }
}
```
迭代解法的时间复杂度为Ｏ(n)。

### 尾递归
其实除了上面普通递归之外，还有一种 [尾递归](https://en.wikipedia.org/wiki/Tail_call)，它的执行效率和所需的资源类似于迭代, 代码如下：
```cpp
int fibo(int a, int b, int n)
{
    if (n < 1) {
        return 0;
    }
    else if (n <= 2) {
        return 1;
    }
    else if (3 == n) {
        return a + b;
    }
    else {
        return fibo2(b, a + b, n - 1);
    }
}
```
这里关于尾递归的问题，自己理解的还不够，先挖个坑，以后再写一篇关于尾递归的。

### 矩阵运算

根据以下公式:

{{<keepit>}}
$$
\begin{bmatrix}
f(n-1) \
f(n)
\end{bmatrix}
=
\begin{bmatrix}
f(n-1) \
f(n-1) + f(n-2)
\end{bmatrix}
=
\begin{bmatrix}
0 \times f(n-2) + 1 \times f(n-1)\
1 \times f(n-2) + 1 \times f(n-1)
\end{bmatrix}
=
\begin{bmatrix}
0 & 1\
1 & 1
\end{bmatrix}
\times
\begin{bmatrix}
f(n-2) \
f(n-1)
\end{bmatrix}
=
\begin{bmatrix}
0 & 1\
1 & 1
\end{bmatrix}^{n-1}
\times
\begin{bmatrix}
f(0) \
f(1)
\end{bmatrix}
$$
{{</keepit>}}

所以求Fibonacci的第n的数就可以简化为$\begin{bmatrix} 0 & 1\\ 1 & 1 \end{bmatrix}^{n-1}$与$\begin{bmatrix} 0\\ 1\end{bmatrix}$相乘。

具体实现的代码如下：
```cpp
class Matrix
{
public:
    Matrix(int height, int width)
        : height_(height),
          width_(width),
          container_(height*width, 0)
    {
    }

    int get(int row, int col)
    {
        assert(row < height_ && col < width_);
        return container_[row*width_ + col];
    }

    void set(int row, int col, int val)
    {
        assert(row < height_ && col < width_);        
        container_[row*width_ + col] = val;
    }
private:
    int height_;
    int width_;
    std::vector<int> container_;
};

Matrix matrix_22_mul(Matrix a, Matrix b)
{
    Matrix result(2, 2);

    result.set(0, 0, a.get(0, 0)*b.get(0, 0) + a.get(0, 1)*b.get(1, 0));
    result.set(0, 1, a.get(0, 0)*b.get(0, 1) + a.get(0, 1)*b.get(1, 1));
    result.set(1, 0, a.get(1, 0)*b.get(0, 0) + a.get(1, 1)*b.get(1, 0));
    result.set(1, 1, a.get(1, 0)*b.get(0, 1) + a.get(1, 1)*b.get(1, 1));

    return result;
}

int fibo(int n)
{
    if (n < 1)
    {
        return 0;
    }
    else if (n == 1)
    {
        return 1;
    }
    else
    {
        Matrix result(2, 2);
        result.set(0, 0, 0);
        result.set(0, 1, 0);
        result.set(1, 0, 1);
        result.set(1, 1, 0);

        Matrix factor(2, 2);
        factor.set(0, 0, 0);
        factor.set(0, 1, 1);
        factor.set(1, 0, 1);
        factor.set(1, 1, 1);

        Matrix multiplier(factor);

        while (n > 2)
        {
            multiplier = matrix_22_mul(multiplier, factor);
            n--;
        }

        result = matrix_22_mul(multiplier, result);

        return result.get(1, 0);
    }
}
```
这一解法跟之前的比看起来是更麻烦了, 但是n次方运算是可以用O(logN)的复杂度解决的, 也叫做矩阵快速幂, 简单来说，就是

{{<keepit>}}
$$
A^n = \begin{cases}
A^{n/2} \times A^{n/2} & \text{if n is even} \
A \times A^{(n-1)/2} \times A^{(n-1)/2} & \text{if n is odd}
\end{cases}
$$
{{</keepit>}}

代码实现如下:
```cpp
if (n < 1)
{
    return 0;
}
else if (n == 1)
{
    return 1;
}
else
{
    Matrix result(2, 2);
    result.set(0, 0, 0);
    result.set(0, 1, 0);
    result.set(1, 0, 1);
    result.set(1, 1, 0);

    Matrix factor(2, 2);
    factor.set(0, 0, 0);
    factor.set(0, 1, 1);
    factor.set(1, 0, 1);
    factor.set(1, 1, 1);

    Matrix multiplier(2, 2);
    multiplier.set(0, 0, 1);
    multiplier.set(0, 1, 0);
    multiplier.set(1, 0, 1);
    multiplier.set(1, 1, 0);

    while (n > 1)
    {
        if (n & 1)
        {
            multiplier = matrix_22_mul(multiplier, factor);
        }

        factor = matrix_22_mul(factor, factor);
        n >>= 1;
    }

    multiplier = matrix_22_mul(multiplier, factor);

    result = matrix_22_mul(multiplier, result);

    return result.get(1, 0);
}
```

### 通项公式法
Fibonacci是有通项公式的, 如下, 具体推导过程请参考引用条目

{{<keepit>}}
$$
f(n) =\frac{(\frac{1 + \sqrt5}{2})^n - (\frac{1 - \sqrt5}{2})^n}{\sqrt5}
$$
{{</keepit>}}

其代码实现如下:
```cpp
int fibo(int n)
{
    return (std::pow((1 + std::sqrt(5)) / 2, n) - std::pow((1 - std::sqrt(5)) / 2, n)) / std::sqrt(5);
}
```
这里从我们的代码实现来看是O(1)的,但其实这里是有计算$x^n$的,这个幂的计算最终的实现大概率还是O(logN)的。而这里引入了浮点运算以及本身的精度问题，所以并不是一个很好的选择。仅供参考。

### 其他
其实SICP这本书里还提供了一种解法, 但实际上和矩阵法是类似的,这里就不具体列出了,可以参考引用7

### Reference
1. [ICS 161: Design and Analysis of Algorithms
Lecture notes for January 9, 1996](https://www.ics.uci.edu/~eppstein/161/960109.html)
2. [斐波那契数列的N种解法](https://zhuanlan.zhihu.com/p/74751385)
3. [几种斐波那契数列项算法的复杂度分析](https://blog.mottomo.moe/categories/Tech/Coding/zh/2019-04-07-Fibonacci-Implementations/)
4. [斐波那契数列数列的三种时间复杂度的实现方法](https://blog.csdn.net/u012684062/article/details/76330075)
5. [计算斐波纳契数，分析算法复杂度](https://blog.gocalf.com/calc-fibonacci)
6. [SICP 第一章习题1.13](https://sicp.readthedocs.io/en/latest/chp1/13.html)
7. [SICP 第一章习题1.19](https://sicp.readthedocs.io/en/latest/chp1/19.html)
8. [leetcode 949](https://www.lintcode.com/problem/949/)