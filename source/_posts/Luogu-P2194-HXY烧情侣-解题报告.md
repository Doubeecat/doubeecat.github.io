---
title: Luogu P2194 HXY烧情侣 解题报告
categories: 解题报告
tags: [图论,Tarjan]
date: 2019-10-11 14:55:00
---
## 题目链接：

[P2194 HXY烧情侣](<https://www.luogu.org/problem/P2194>)

## 题目描述：

 > 众所周知，HXY已经加入了FFF团。现在她要开始喜（sang）闻（xin）乐（bing）见（kuang）地烧情侣了。这里有$n$座电影院，$n$对情侣分别在每座电影院里，然后电影院里都有汽油，但是要使用它需要一定的费用。$m$条单向通道连接相邻的两对情侣所在电影院。然后HXY有个绝技，如果她能从一个点开始烧，最后回到这个点，那么烧这条回路上的情侣的费用只需要该点的汽油费即可。并且每对情侣只需烧一遍，电影院可以重复去。然后她想花尽可能少的费用烧掉所有的情侣。问最少需要多少费用，并且当费用最少时的方案数是多少？由于方案数可能过大，所以请输出方案数对$1e9+7$取模的结果。

> （注：这里HXY每次可以从任何一个点开始走回路。就是说一个回路走完了，下一个开始位置可以任选。所以说不存在烧不了所有情侣的情况，即使图不连通，HXY自行选择顶点进行烧情侣行动。且走过的道路可以重复走。）
<!--more-->
## 解题思路：

题目中写的很清楚，这题直接套一个 Tarjan 就完了

现在讨论一下两个问题：

1. 最少需要多少费用

   这个比较好想，只要将缩点后的每个 SCC 权值变成其中点权最小的即可

2. 总共有多少方案

   这个比较难想，但是我们可以结合一下乘法原理的思想，将每个 SCC 中的点权等于 SCC 的点权的个数相乘

可能有点绕口，我们用形式化的语言来描述一下

记 $siz_i$  为 $SCC_i$ 中点权最小的权值， $num_i$ 为 $SCC_i$ 中点权等于最小点权的权值的点，$cnt$ 为 SCC 数量

则有

$$ans_1 = \sum^{cnt}_{i=1}siz[i]$$

$$ans_2 = \prod^{cnt}_{i=1}num_i$$

便可写出代码

## 代码：

```cpp
#include <cstring>
#include <cstdio>
#include <cctype>
#include <cmath>
#include <stack>
using std::stack;
#define long long ll
#define register int ri

char buf[1<<21],*p1=buf,*p2=buf;
inline int gc(){return p1==p2&&(p2=(p1=buf)+fread(buf,1,1<<21,stdin),p1==p2)?EOF:*p1++;}
inline int read() {
	char v = gc();int x = 0,f = 1;
	while (!isdigit(v)) {if (v == '-')f = -1;v = gc();}
	while (isdigit(v)) {x = x * 10 + v - 48;v = gc();}
	return x * f;
}
const int N = 100001;
const int M = 500001;
const int INF = 0x3f3f3f3f;

int to[M],hd[N],nxt[M],tot;

inline void add(int u,int v) {to[++tot] = v;nxt[tot] = hd[u];hd[u] = tot;}
inline void addedge(int u,int v) {add(u,v),add(v,u);}

template <typename T> inline T min(T x,T y) {return x<y?x:y;}
template <typename T> inline T max(T x,T y) {return x>y?x:y;}

int dfn[N],low[N],c[N],cnt,num,siz[N],p[N],pre[N],ins[N],n,m,ans1,ans2 = 1;
stack <int> s;

void tarjan(int x) {
	dfn[x] = low[x] = ++cnt;
	s.push(x);ins[x] = 1;
	for (int i = hd[x];i;i = nxt[i]) {
		if (!dfn[to[i]]) {
			tarjan(to[i]);
			low[x] = min(low[x],low[to[i]]);
		}
		else if (ins[to[i]]) {
			low[x] = min(low[x],dfn[to[i]]);
		}
	}
	if (dfn[x] == low[x]) {
		int y;c[x] = ++num;
		do {
			y = s.top();c[y] = num;ins[y] = 0;s.pop();
			if (pre[y] < p[num]) {
				p[num] = pre[y];
				siz[num] = 1;
			}
			else if (pre[y] == p[num]) {
				++siz[num];
			}
		}while (x != y);
	}
}

signed main() {
	memset(p,INF,sizeof(p));
	n = read();
	for (int i = 1;i <= n;++i) pre[i] = read();
	m = read();
	for (int i = 1;i <= m;++i) {
		int x = read(),y = read();add(x,y);
	}
	for (int i = 1;i <= n;++i) {
		if (!dfn[i]) tarjan(i);
	}
	for (int i = 1;i <= num;++i) {
		ans1 += p[i];ans2 *= siz[i];
	}
	printf("%d %d",ans1,ans2);
	return 0;
}
```

