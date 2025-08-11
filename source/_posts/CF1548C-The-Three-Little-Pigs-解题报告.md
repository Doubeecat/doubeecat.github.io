title: CF1548C The Three Little Pigs 解题报告
categories: 解题报告
tags: [DP,计数问题]
date: 2021-08-23 16:54:00
---
> 给定一个 $n$ 和 $q$，有 $q$ 次询问，每次询问给定一个 $x$。
>
> 要求你求出 $\sum\limits_{i =1}^{n} \binom{3i}{x}$
>
> $1 \leq n \leq 10^6,1 \leq q \leq 2\times 10^5,x \leq 3n$

~~感谢学弟让我这只千年大鸽子第一次有如此热情熬夜赶工~~

<!--more-->

首先观察题目这个式子，我们发现，实际上因为是  $3i$ ，我们可以分成三类来计算：$3i,3i+1,3i+2$ 这样会方便我们的思考。

自然的，直接求和如果没有一些科技是不太现实的，我们考虑 DP。

设 $f_{i,k}$ 表示 $\sum\limits_{i = 0}^{n-1}\binom{3i + k}{x}$ ，这样定义的好处在于，我们发现答案可以直接表示为  $\binom{3n}{x} + f_{n-1,0}$.

接下来我们考虑怎么计算 $f_{x,k}$，我们回想一下有组合恒等式 
$$
\displaystyle{ \sum^n_{i=r}{i\choose r}={n+1\choose r+1} }
$$

也就是说，有
$$
\displaystyle{ \sum^{3n-1}_{i=0}{i\choose x}= \sum\limits_{i = 0}^{n-1}\binom{3i}{x} + \sum\limits_{i = 0}^{n-1}\binom{3i + 1}{x} + \sum\limits_{i = 0}^{n-1}\binom{3i + 2}{x} = {3n\choose x+1} }
$$
变成我们刚才的定义就得到了一个优美的式子
$$
\displaystyle{f_{i,0} + f_{i,1} + f_{i,2} = {3i\choose x+1}}
$$
我们得到了一个关系式，但是这远远不够，我们需要发掘更多性质。

由帕斯卡公式 $\binom{n-1}{m}+\binom{n-1}{m-1} = \binom{n}{m}$ 我们可以得到
$$
\sum_{i=0}^{n-1} {3i \choose x} + \sum_{i=0}^{n-1} {3i \choose x-1} = \sum_{i=0}^{n-1} {3i+1 \choose x}
$$
我们可以将 $i = 0,1$ 带入，得到下面两个式子

$$
f_{i,1} = f_{i-1,0} + f_{i,0} \\
f_{i,2} = f_{i-1,1} + f_{i,1}
$$

至此，我们发现上面的三个式子组成了一个三元一次方程组，此时我们可以换元，尝试解出 $f_{i,0}$ 直接的递推式。

把后面两个式子都代回第一个式子中易得

$$
3f_{i,0}={3i\choose x+1} - 2f_{i-1,0} - f_{i-1,1} \\
f_{i,0} = \frac{{3i\choose x+1} - 2f_{i-1,0} - f_{i-1,1}} {3}
$$

所以，我们就得到了总的递推公式：
$$
\left\{
	\begin{aligned} 
	f_{i,0} &= \frac{{3i\choose x+1} - 2f_{i-1,0} - f_{i-1,1}} {3} \\
	f_{i,1} &= f_{i-1,0} + f_{i,0} \\
	f_{i,2} &= f_{i-1,1} + f_{i,1}
	\end{aligned}
\right.
$$


边界条件为 $f_{0,0} = f_{0,1} = f_{0,2} = n$

对于 $1$ 到 $3n$ 的 $f_{i,0/1/2}$ 预处理一遍，注意需要使用线性求逆元，否则可能被卡常，时间复杂度 $O(n+q)$