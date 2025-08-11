title: Luogu P4552 [Poetize6] IncDec Sequence 解题报告
categories: 解题报告
tags: [差分]
date: 2020-05-14 13:36:00
---
> [P2132 IncDec Sequence](https://www.luogu.com.cn/problem/P4552)
>
> 给定一个长度为 $n$ 的数列 ${a_1,a_2,\cdots,a_n}$，每次可以选择一个区间$[l,r]$，使这个区间内的数都加 $1$ 或者都减 $1$。 
  
> 请问至少需要多少次操作才能使数列中的所有数都一样，并求出在保证最少次数的前提下，最终得到的数列有多少种。

<!--more-->

一道有意思的差分题目，第一个问题相当于积木大赛，可以直接分析也可以根据差分性质来搞。  
首先做 $a$ 数列的差分数列 $b$，那么问题就变成了使得$b_2...b_n = 0$  
首先考虑配对一个正数一个负数 这样操作次数是最少的 首先考虑这种 
这种的操作次数为 $\min(p,q)$ ($p,q$为差分数列正数和，负数和)

显然，这样会造成最后至多一个数没有消灭，此时可以将它与 $b_1$ 或者 $b_{n+1}$ 配对。
由于两种配对方式不同，会出现$|p-q|+1$种序列，并且最小次数为$|p-q|$
于是就解决了本题。
```cpp
#include <cstring>
#include <cstdio>
#include <cctype>
#include <cmath>
#define ll long long
#define ri register int

char buf[1 << 20], *p1, *p2;
#define getchar() (p1 == p2 && (p2 = (p1 = buf) + fread(buf, 1, 1 << 20, stdin), p1 == p2)?EOF: *p1++)
template <typename T> inline void read(T &t) {
	ri v = getchar();T f = 1;t = 0;
	while (!isdigit(v)) {if (v == '-')f = -1;v = getchar();}
	while (isdigit(v)) {t = t * 10 + v - 48;v = getchar();}
	t *= f;
}
template <typename T,typename... Args> inline void read(T &t,Args&... args) {
		read(t);read(args...);
	}
const int N = 100010;

template <typename T> inline T min(T x,T y) {return x<y?x:y;}
template <typename T> inline T max(T x,T y) {return x>y?x:y;}

ll a[N],b[N],n,p,q;
int main() {
    read(n);
    for (int i = 1;i <= n;++i) read(a[i]);
    for (int i = 2;i <= n;++i) {b[i] = a[i]-a[i-1];if (b[i] > 0)p+=b[i];else q -=b[i];}
    printf("%lld\n%lld",max(p,q),abs(p-q)+1);
    
    
    
}

```