---
title: gcd(未完成版)
categories: 学习笔记
tags: [数论]
date: 2021-04-10 15:27:00
---

> $\gcd$ 是出现在数论而又披着同余的外衣的唯一的函数。

因为关于 $\gcd$ 的东西实际上非常多非常杂，写一个博客记录一下。

而且数论里面基本上是个东西就离不开 $\gcd$ 所以这也是个非常重要的东西。

<!--more-->

---

# 1.基础计算

常用欧几里得算法计算，也就是我们经常用的 $\gcd(a,b) = \gcd(b,a \bmod b)$

证明（直接抄OI-Wiki的，只是为了不误人子弟，日常谁无聊证明这个啊）：

设 $a=bk+c$，显然有 $c=a \bmod b$。设 $d \mid a,~d \mid b$，则 $c=a-bk, \frac{c}{d}=\frac{a}{d}-\frac{b}{d}k$。

由右边的式子可知 $\frac{c}{d}$ 为整数，即 $d \mid c$ 所以对于 $a,b$ 的公约数，它也会是 $a \bmod b$ 的公约数。

反过来也需要证明：

设 $d \mid b,\mid (a \bmod b)$，我们还是可以像之前一样得到以下式子 $\frac{a\bmod b}{d}=\frac{a}{d}-\frac{b}{d}k,\frac{a\bmod b}{d}+\frac{b}{d}k=\frac{a}{d}$。

因为左边式子显然为整数，所以 $\frac{a}{d}$ 也为整数，即 $d \mid a$，所以 $b,a\bmod b$ 的公约数也是 $a,b$ 的公约数。

既然两式公约数都是相同的，那么最大公约数也会相同。

所以得到式子 $\gcd(a,b)=\gcd(b,a\bmod b)$

## Code

```cpp
int gcd(int a,int b) {return !b ? a : gcd(b,a % b);}
```

# 2.裴蜀定理

设 $\gcd(a, b) = d$，则对任意整数 $x, y$，有 $d|(ax + by)$ 成立

特别地，一定存在 $x, y$ 满足 $ax + by = d$

等价的表述：不定方程 $ax + by = c (a,b,c \in  \mathbb{Z})$有解的充要条件为 $(a, b)|c$

推论：$a, b$ 互质等价于 $ax + by = 1$ 有解

### 证明

显然有 $d | a,d | b$

根据之前的引理有 $d | ax,d | by$

故 $d | (ax+by)$

对于第二个结论（

若任何一个等于 $0$ , 则 $\gcd(a,b)=a$ . 这时定理显然成立。

若 $a,b$ 不等于 $0$ .

由于 $\gcd(a,b)=\gcd(a,-b)$ ,

不妨设 $a,b$ 都大于 $0$ , $a\geq b,\gcd(a,b)=d$ .

对 $ax+by=d$ , 两边同时除以 $d$ , 可得 $a_1x+b_1y=1$ , 其中 $(a_1,b_1)=1$ .

转证 $a_1x+b_1y=1$ .

我们先回顾一下辗转相除法是怎么做的，由 $\gcd(a, b) \rightarrow \gcd(b,a\bmod b) \rightarrow ...$ 我们把模出来的数据叫做 $r$ 于是，有

$$ \gcd(a_1,b_1)=\gcd(b_1,r_1)=\gcd(r_1,r_2)=\cdots=(r_{n-1},r_n)=1 $$

把辗转相除法中的运算展开，做成带余数的除法，得
$$
\begin{aligned}
    a_1 &= q_1b+r_1 &(0\leq r_1<b_1) \\
    b_1 &= q_2r_1+r_2 &(0\leq r_2<r_1) \\ 
    r_1 &= q_3r_2+r_3 &(0\leq r_3<r_2) \\ 
    &\cdots \\
    r_{n-3} &= q_{n-1}r_{n-2}+r_{n-1} \\
    r_{n-2} &= q_nr_{n-1}+r_n \\
    r_{n-1} &= q_{n+1}r_n
\end{aligned} 
$$
不妨令辗转相除法在除到互质的时候退出则 $r_n=1$ 所以有（ $q$ 被换成了 $x$ ，为了符合最终形式）

$$ r_{n-2}=x_nr_{n-1}+1 $$

即

$$ 1=r_{n-2}-x_nr_{n-1} $$

由倒数第三个式子 $r_{n-1}=r_{n-3}-x_{n-1}r_{n-2}$ 代入上式，得

$$ 1=(1+x_nx_{n-1})r_{n-2}-x_nr_{n-3} $$

然后用同样的办法用它上面的等式逐个地消去 $r_{n-2},\cdots,r_1$ ,

可证得 $1=a_1x+b_1y$ . 这样等于是一般式中 $d=1$ 的情况。

# 3.Exgcd

重头戏，也是写这篇博客的原因之一

由裴蜀定理可得，方程 $ax + by = \gcd(a,b)$ 必有一组解。

所以我们考虑如何求出这样的方程的一组解。

## 证明：

设 
$$
\begin{aligned}
    &ax_1 + by_1 = \gcd(a,b) \\
    &bx_2 + (a \bmod b)y_2 = \gcd(b,a \bmod b) 
\end{aligned}
$$
则
$$
\begin{aligned}
    &\because \gcd(a,b) = \gcd(b,a \bmod b)\\
    &\therefore ax_1 + by_1 = bx_2 + (a \bmod b)y_2 \\
    &\because (a \bmod b ) = a - (\lfloor \frac{a}{b} \rfloor \times b)\\
    &\therefore ax_1 + by_1 = bx_2 + (a-\lfloor \frac{a}{b} \rfloor \times b) y_2 \\
\end{aligned}
$$
展开得
$$
\begin{aligned}
    &ax_1 + by_1 = ay_2 + bx_2 - \lfloor \frac{a}{b} \rfloor by_2 = &ay_2 + b(x_2 - \lfloor \frac{a}{b} \rfloor y_2)\\
    &\because a = a,b = b \\
    &\therefore x_1 = y_2 , y_1 = x_2 - \lfloor \frac{a}{b} \rfloor y_2
\end{aligned}
$$
所以就可以递归求解了。

## Code

```cpp
ll exgcd(ll a,ll b,ll& x,ll& y) {
	ll ret,tmp;
	if (!b) {
		x = 1,y = 0;
		return a;
	}
	ret = exgcd(b,a%b,x,y);
	tmp = x,x = y,y = tmp -a/b * y;
	return ret; 
}
```

### 拓展

考虑形如 $ax + by = c (a,b,c \in  \mathbb{Z})$ 的方程的所有解

首先由裴蜀定理得，不定方程 $ax + by = c (a,b,c \in  \mathbb{Z})$有解的充要条件为 $(a, b)|c$ 。

先考虑求一组特殊解的情况：首先将 $a,b,c$ 都除以 $\gcd(a,b)$ 得 $a_0x + b_0y = c_0$

此时 $\gcd(a_0,b_0) = 1$,所以我们可以求出 $a_0x + b_0y = 1$ 的一组解 $x_0,y_0$ ，则原方程的一组特解为

$$x_1 = \frac{x_0c}{\gcd(a,b)},y_1 = \frac{y_0c}{\gcd (a,b)}$$

然后我们考虑构造所有解的形式：

设有 $d (d \in \mathbb{Q})$ 那么必有 $a(x_1 + db) + b(y_1 - da) = c$

同时，这组解需要保证 $db,da \in  \mathbb{Z}$

令当 $d$ 取到最小可能的正值的 $d_x = db,d_y = da$ ,那么任意解中这两个变量与 $x_1,x_2$ 的偏差一定分别是 $d_x,d_y$ 的倍数。

显然，最小的时候 $d_x = \frac{b}{\gcd (a,b)},d_y = \frac{-a}{\gcd (a,b)}$

所有解就是 $x = x_0 + kd_x,y = y_0 + kd_y (k \in  \mathbb{Z})$

例题：[P5656 【模板】二元一次不定方程 (exgcd)](https://www.luogu.com.cn/problem/P5656)

------

# 4.$\varphi(n)$

欧拉函数 $\varphi(n)$，表示 $1 \sim n-1$ 里和 $n$ 互质的数的个数。

## 性质

它有以下重要性质：

### 性质1：当 $x \perp y$ 时， $\varphi(xy) = \varphi(x) \varphi(y)$

证明：对于每个 $z \nmid xy$，必有 $z = ab$，其中 $a\nmid x,b\nmid y$。所以就比较显然了。

### 性质2：当 $p$ 为素数，$\varphi(p^k) = p^k - p^{k-1}$

证明：考虑每个 $x | p^k$ 有 $x = ap$。这样的 $a$ 在 $1 \sim p^k - 1$ 里面有 $p^{k-1} - 1$ 个，所以易得。

### 性质3：