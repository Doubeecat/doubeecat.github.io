title: CF1648B Integral Array 解题报告
categories: 解题报告
tags: []
date: 2022-07-20 02:50:00
---
> CF1648B Integral Array
> 
> 给定一个数组 $a $ ,  我们称该数组完整需要满足 ：若数组 $a$ 中存在两数 $x,y $, 使 $y \le x$ ($x,y$ 可以是同一个数) , 则 $\left\lfloor\dfrac{x}{y}\right\rfloor$ 也必须在数组 $\ a$ 中 , 现需要判断数组$\ a$ 是否完整 。
> 
> $T \le 10^4 ,\sum n\le 10^6,\sum c\le 10^6$ ,  其中$\ T$ 为数据组数  ， $\ n$ 为$\ a$ 的元素个数，满足 $a$ 中元素 $\le c$。

<!--more-->

## 分析

直接枚举 $x,y$ 的话的时间复杂度是 $\mathcal{O}(n^2)$ 的，显然不行。

所以我们考虑如何优化这个枚举。

> 引理1
> $$
 \forall n \in \mathbb{N}_{+}, \left|\left\{ \lfloor \frac{n}{d} \rfloor \mid d \in \mathbb{N}_{+},d\leq n \right\}\right| \leq \lfloor 2\sqrt{n} \rfloor 
> $$
> 简略证明：
> 
> 对于 $d\leq \left\lfloor\sqrt{n}\right\rfloor$ 来说，$\left\lfloor\frac{n}{d}\right\rfloor$ 有 $\left\lfloor\sqrt{n}\right\rfloor$ 种取值
> 
> 对于 $d> \left\lfloor\sqrt{n}\right\rfloor$ 来说，有 $\left\lfloor\frac{n}{d}\right\rfloor\leq\left\lfloor\sqrt{n}\right\rfloor$ 同样只有 $\left\lfloor\sqrt{n}\right\rfloor$ 种取值
> 
> (from [OI-wiki](https://oi-wiki.org/math/number-theory/sqrt-decomposition/#2))

根据引理 1，我们知道了实际上不同的 $\left\lfloor\frac{x}{y}\right\rfloor$ 的个数为 $2\left\lfloor\sqrt{x}\right\rfloor$，所以我们首先考虑枚举 $\left\lfloor\frac{x}{y}\right\rfloor = k \leq \left\lfloor\sqrt{x}\right\rfloor$ 并且进行判断。

判断的部分，我们可以开一个桶来储存每一个数字的出现个数，对这个桶做前缀和就可以很方便的在 $\mathcal{O}(1)$ 的时间内查询 $[l,r]$ 是否存在一个数。

但是这样的时间复杂度是 $\mathcal{O}(n\sqrt{n})$ 的，还是无法通过本题，如何进一步优化呢？

我们换一种思路考虑，采用一个类似线性筛的思路。同样是枚举 $\left\lfloor\frac{x}{y}\right\rfloor= k\times i$，其中 $x | i$。检查 $[ki,(k+1)i)$（注意是左开右闭）中是否存在数，如果存在，那么 $i$ 一定要存在，否则不满足。

根据调和级数相关有 $\sum\limits_{i = 1}^n = \ln n$，故时间复杂度为 $\mathcal{O}(n\ln n)$ 可以通过本题。

## 代码

```cpp
const int N = 2e6  +10000;

int n,c,a[N],buc[N],pre[N];

void solve() {
    read(n,c);
    for (int i = 1;i <= 2*c;++i) pre[i] = 0,buc[i] = 0;//注意初始化到 2 * c
    for (int i = 1;i <= n;++i) read(a[i]),buc[a[i]]++;
    for (int i = 1;i <= 2*c;++i) pre[i] = pre[i - 1] + buc[i];
    bool flag = 0;
    for (int i = 1;i <= c;++i) {
        if (!buc[i]) continue;
        for (int j = 1;i * j <= c;++j) {
            int l = i * j,r = i * (j + 1) - 1;
            if (!buc[j]) {
                if (pre[r] - pre[l - 1]) {
                    flag = 1;
                    break;
                }
            }
        }
    }
    if (!flag) puts("Yes");
    else puts("No");
}

signed main() {
    int T;read(T);
    while (T--) solve();
	return 0;
}
```