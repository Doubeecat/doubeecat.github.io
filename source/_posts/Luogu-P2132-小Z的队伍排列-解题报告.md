---
title: Luogu P2132 小Z的队伍排列 解题报告
categories: 解题报告
tags: [DP,线性DP]
date: 2020-06-20 12:34:14
---


> [P2132 小Z的队伍排列](<https://www.luogu.com.cn/problem/P2132>)
>
> 小Z想给班里的同学拍一张合影，为此需要先让大家排好队伍。他希望大家站成 $k$ 排，并规定了每排的人数，保证每一排的人数都不多于后面一排的人数。
>
> 这时小Z发现队伍看起来还是乱糟糟的，原因是大家的身高互不相同。于是，他希望排头对齐，每位同学都比自己正后方的同学以及排头方向的同学矮。
>
> 排完以后，善于思考的小Z还想知道一共有多少种排法。

<!--more-->
## 解题思路：

线性 DP。

首先因为“保证每一排的人数都不多于后面一排的人数”且“每位同学都比自己正后方的同学以及排头方向的同学矮”，所以合法方案中每行每列的身高显然单调。

所以我们可以从高到低考虑每个学生所站的位置，这样搞的话，我们只要用一个$(a_1,a_2,a_3...a_n)$ 就能体现出已经处理的东西的轮廓。

而每次考虑一个学生的时候，为了保证单调性，我们只能将他放在满足人员未满并且 $a_i < a_{i-1}$ 或 $i = 1$ 的行中。

所以定义 $(a_1,a_2,a_3...a_n)$ 为阶段，这样就满足了动态规划问题的最优子结构性质，并且每安排一名新学生时候 $a_1,a_2,a_3...a_n$ 中总有一个会 $+1$，满足各个维度线性增长的原则，也就是线性 DP 了。

接下来，我们来设计状态转移方程，本题中的 $k \leq 5$，所以~~无脑~~设计 

$f_{a_1,a_2,a_3,a_4,a_5}$表示每排从最左边起分别站了 $a_1,a_2,a_3,a_4,a_5$ 个人的方案数量

$f_{0,0,0,0,0} = 1$ 

目标：$f_{k_1,k_2,k_3,k_4,k_5}$

转移：

当 $i = 1$ 时，如果 $a_1 < k_1$，也就是人还没被放满时，$f_{a_1+1,a_2,a_3,a_4,a_5}+=f_{a_1,a_2,a_3,a_4,a_5}$

当 $i > 1$ 时，如果 $a_i < k_i \& a_i < a_{i-1}$，$f_{a_1,a_2+1,a_3,a_4,a_5}+=f_{a_1,a_2,a_3,a_4,a_5}$

（此处为当$i = 2$时，其他同理。）

## 代码：

```cpp
#include <cstring>
#include <iostream>
#include <cstdio>
#include <cctype>
#include <string>
#include <cmath>
#include <stack>
using namespace std;

#define ll long long
#define ri register int

char buf[1 << 20], *p1, *p2;
//#define getchar() (p1 == p2 && (p2 = (p1 = buf) + fread(buf, 1, 1 << 20, stdin), p1 == p2)?EOF: *p1++)
template <typename T> inline void read(T &t) {
	ri v = getchar();T f = 1;t = 0;
	while (!isdigit(v)) {if (v == '-')f = -1;v = getchar();}
	while (isdigit(v)) {t = t * 10 + v - 48;v = getchar();}
	t *= f;
}
template <typename T,typename... Args> inline void read(T &t,Args&... args) {
		read(t);read(args...);
	}
const int INF = 0x3f3f3f3f;
const int N = 31;

ll f[N][N][N][N][N],n;
int a[N];

signed main() {
	read(n);
	for (int i = 1;i <= n;++i) read(a[i]);
	f[0][0][0][0][0] = 1;
	for (int a1 = 0;a1 <= a[1];++a1) {
		 for (int a2 = 0;a2 <= min(a1,a[2]);++a2) {
		 	for (int a3 = 0;a3 <= min(a2,a[3]);++a3) {
		 		for (int a4 = 0;a4 <= min(a3,a[4]);++a4) {
		 			for (int a5 = 0;a5 <= min(a4,a[5]);++a5) {
		 				ll &p = f[a1][a2][a3][a4][a5];//指针储存，传统艺能
		 				if (a1 && a1 > a2) p += f[a1-1][a2][a3][a4][a5];
		 				if (a2 && a2 > a3) p += f[a1][a2-1][a3][a4][a5];
		 				if (a3 && a3 > a4) p += f[a1][a2][a3-1][a4][a5];
		 				if (a4 && a4 > a5) p += f[a1][a2][a3][a4-1][a5];
		 				if (a5) p += f[a1][a2][a3][a4][a5-1];
		 			}
		 		}
		 	}
		 }
	}
	printf("%lld\n", f[a[1]][a[2]][a[3]][a[4]][a[5]]);
	return 0;
}
```

