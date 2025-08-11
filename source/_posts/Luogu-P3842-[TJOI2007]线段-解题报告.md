---
title: Luogu P3842 [TJOI2007]线段 解题报告
categories: 解题报告
tags: [DP,线性DP]
date: 2020-06-27 13:50:34
---
> [P3842 [TJOI2007]线段]([<https://www.luogu.com.cn/problem/P2132>](https://www.luogu.com.cn/problem/P3842))
>
> 在一个 $n*n$ 的平面上，在每一行中有一条线段，第 $i$ 行的线段的左端点是$(i, l_i)$，右端点是$(i, r_i)$，其中 $i \leq l_i \leq r_i \leq n$。
>
> 你从$(1, 1)$点出发，要求沿途走过所有的线段，最终到达$(n, n)$点，且所走的路程长度要尽量短。
> 
> 更具体一些说，你在任何时候只能选择向下走一步（行数增加 1）、向左走一步（列数减少 1）或是向右走一步（列数增加 1）。当然，由于你不能向上行走，因此在从任何一行向下走到另一行的时候，你必须保证已经走完本行的那条线段。

<!--more-->
## 解题思路：
线性 DP.

首先我们观察题目中的三种操作，可以概括成：

- 在横坐标上 +1 -1
- 在纵坐标上 +1

初步定义 $f_{i,j}$ 代表在第 $i$ 行第 $j$ 列里的最短长度。

然而我们很快可以发现，这里的 $n \leq 20000$ ，而且转移时并不是很好写，有许多冗余状态（横坐标在最长线段外的点我们显然不可能走到）

我们再对这个过程进行深入分析，发现其实在走线段的过程中只有横坐标会变化，而且这个变化并不用 DP 维护，暴力加上即可。

所以我们再定义 $f_{i,0/1}$ 代表在第 $i$ 行时，若为 0，则说明**走完这条线段后**在线段的左端点，若为 1，则说明**走完这条线段后**在线段的右端点。

这样我们的转移也呼之欲出了，

$f_{i,0}$ 可以从 $f_{i-1,0}$ 或 $f_{i-1,1}$ 转移来，转移方程为

$$f_{i,0} = \min(f_{i-1,0} + \operatorname{dis}(l_{i-1},r_i),f_{i-1,1} + \operatorname{dis}(r_{i-1},r_i)) + len + 1$$

上述方程表示由上一行的线段的左端点转移来时，先走到当前行的右端点再向下走，最后走一遍到当前线段的左端点。右端点同理。

同理，

$f_{i,1}$  可以从 $f_{i-1,0}$ 或 $f_{i-1,1}$ 转移来，转移方程为

$$f_{i,1} = \min(f_{i-1,0} + \operatorname{dis}(l_{i-1},l_i),f_{i-1,1} + \operatorname{dis}(r_{i-1},l_i)) + len + 1$$

上述方程表示由上一行的线段的左端点转移来时，先走到当前行的左端点再向下走，最后走一遍到当前线段的左端点。右端点同理。

因为其满足各个维度线性增长，故其为线性 DP ，时间复杂度 $O(n)$
## 代码：
```cpp
#include <algorithm>
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

template <typename T> inline T min(T x,T y) {return x<y?x:y;}
template <typename T> inline T max(T x,T y) {return x>y?x:y;}
const int INF = 0x3f3f3f3f;
const ll inf = 0x3f3f3f3f3f3f3f3fLL;
const int N = 25100;

int f[N][2],l[N],r[N],n;

inline int dis(int x,int y) {return abs(x-y);}

signed main() {
	read(n);
	for (int i = 1;i <=  n;++i) read(l[i],r[i]);
	f[1][1] = dis(1,r[1]),f[1][0] = dis(1,r[1]) + dis(l[1],r[1]);
	for (int i = 2;i <= n;++i) {
		int len = dis(l[i],r[i]);
		f[i][0] = min(f[i-1][0]+dis(l[i-1],r[i]),f[i-1][1]+dis(r[i-1],r[i])) + len + 1;
		f[i][1] = min(f[i-1][0]+dis(l[i-1],l[i]),f[i-1][1]+dis(r[i-1],l[i])) + len + 1;
	}
	int ans = min(f[n][0] + dis(l[n],n),f[n][1] + dis(r[n],n));
	printf("%d\n", ans);
	return 0;
}
```


