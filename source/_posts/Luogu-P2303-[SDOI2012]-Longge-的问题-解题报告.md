---\r\title: Luogu P2303 [SDOI2012] Longge 的问题 解题报告
categories: 解题报告
tags: [数学,欧拉函数]
date: 2020-08-05 04:03:24
---

> [P2303 [SDOI2012] Longge 的问题](https://www.luogu.com.cn/problem/P2303)
> 
> 求 $\sum\limits_{i=1}^n \gcd(i, n)$ 
> 
> $n \leq 2 ^ {32}$

<!-- more -->

## 解题思路：

数学题。（第一次推出式子来真的好开心hhhh）

首先看到 gcd ，第一反应应该是从 $n$ 的因数开始思考。

那么我们把 $n$ 质因数分解，设 $d | n$

我们要求的东西其实就是 $\sum\limits_{d = 1}^n (d * \sum\limits_{i = 1}^n[\gcd (i,n) = j])$

化简一波式子：
$$
\begin{aligned}
    &\sum\limits_{d = 1}^n(d * \sum\limits_{i = 1}^n[\gcd (i,n) = j])\\
    & = \sum\limits_{d = 1}^n(d * \sum\limits_{i = 1}^n[\gcd (i/j,n/j)])\\
    & = \sum\limits_{d = 1}^n(d * \varphi(n/j))\\
    & = \sum\limits_{d|n}(d * \varphi(n/j))
\end{aligned}
$$
然后直接算就行了。
时间复杂度$O(d(n) * \sqrt{n})$
## 代码：
```cpp
ll n,ans;

ll phi(int x) {
    if (x == 1) return 1;
    ll ans = x;
    for (int i = 2;i *i <= x;++i) {
        if (x % i == 0) {
            while (x % i == 0) {x /= i;}
            ans = (ans * (i-1) / i);
        }
    }
    if (x > 1) ans = (ans * (x-1) / x);
    return ans;
}

signed main() {
	read(n);
    for (int i = 1;i * i <= n;++i) {
        if (n % i == 0) {
            ans += i*phi(n/i);
            if (i * i != n) ans += n/i * phi(i);
        }
    }
    printf("%lld",ans);
	return 0;
}
```
