---
title: AcWing 102. 最佳牛围栏 解题报告
categories: 学习笔记
tags: []
date: 2020-05-22 13:53:00
---

> 给你一个长度为 $n$ 的数列，求出这个数列的平均数最大且长度不小于$L$的连续子段。 

<!--more-->
## 思路
首先思考一下怎么快速的求一个子序列平均数最大的问题：  
在一个数列$a_1...a_n$中，求一个平均数最大的连续子段。  
我们可以观察到一个性质：设$a$的平均数为$p$  
则$a_i = p + (a_i-p)$，令 $b_i = a_i - p$ 显然有 $\sum^n_{i=1} b_i = 0$  
所以我们可以利用$b_i$来判断是否为真正的平均数。  
若$\sum^n_{i=1} b_i < 0$，说明 $p$ 比真正的平均数大了。
若$\sum^n_{i=1} b_i > 0$，说明 $p$ 比真正的平均数小了。
而我们可以发现，最终的答案 $p$ 具有单调性：
设当前结果为$s$，若$s < p$，一定有更大的平均值
若$s > p$，则它的长度就是错的。
所以我们可以简单的利用上面两个结论进行二分答案。
回归题目，我们发现，如果要让 $p$ 最大，则要找到一个 $\sum^r_{i=l}$ 最大的字段。
问题就被转化为了求一个数列的长度不小于$L$的最大子段和。
$O(n)$ 求最大子段和显然不用我多讲，加上一个限制也并不难推导。
因为当前位置的答案只能由之前$1...i-L$的答案来得出，也就是说，随着$i$的增长，我们的决策集合每次只会增大1，开个变量随便维护下就完了。
这里借用lyd老师的柿子，$sum_i$表示$i$的前缀和。
$$\max_{i-j\geq L} \{a_{j+1}+...+a_i\} = \max_{L\leq i\leq n} \{sum_i - \min_{0\leq j \leq i-L} sum_j\}$$
这里就涵盖了一个化归的思想，巧妙地将难题转化为了一个见过的题目。

## 代码
```cpp
#include <cstring>
#include <iostream>
#include <cstdio>
#include <cctype>
#include <cmath>
#define ll long long
#define ri register int

const double eps = 1e-5;
const int N = 100010;
const int M = 100010;

template <typename T> inline T min(T x,T y) {return x<y?x:y;}
template <typename T> inline T max(T x,T y) {return x>y?x:y;}

int n,m;
double a[N],sum[N],b[N];

bool check(double mid) {
    for (int i = 1;i <= n;++i) {
        b[i] = a[i] - mid;
    }
    for (int i = 1;i <= n;++i) {
        sum[i] = sum[i-1] + b[i];
    }
    double minn = 1e10,maxx = -1e10;
    for (int i = m;i <= n;++i) {
        minn = min(minn,sum[i-m]);
        maxx = max(maxx,sum[i]-minn);
    }
    if (maxx >= 0) return true;
    else return false;
}

int main() {
    std::cin >> n >> m;
    for (int i = 1;i <= n;++i) scanf("%lf",&a[i]);
    double l = -1e6,r = 1e6;
    while (r-l > eps) {
        double mid = (l + r) / 2;
        if (check(mid)) l = mid;
        else r = mid;
    }
    std::cout << int(r*1000);
    return 0;
}
```