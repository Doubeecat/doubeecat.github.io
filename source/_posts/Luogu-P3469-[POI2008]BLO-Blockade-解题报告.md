---
title: Luogu P3469 [POI2008]BLO-Blockade 解题报告
categories: 解题报告
tags: [图论,Tarjan,割点]
date: 2019-09-07 16:17:00
---
> [[POI2008]BLO-Blockade](<https://www.luogu.org/problem/P3469>)

> 有 $n$ 个节点的无向图，定义封锁一个点为切断这个点的所有连边。求每个节点被封锁后图内的不连通有序点对个数。

<!--more-->
## 解题思路：

Tarjan。

首先分类讨论一下，封锁一个点有两种情况：

1. 不是割点

   这种情况好搞，从图中显然可以看出只有自己和其他 $n - 1 $ 个节点不连通，因为是有序节点，所以答案为 $2 \times (n-1)$

   ![](https://s2.ax1x.com/2019/09/08/n3Muo8.png)

2. 是割点

   这种情况就有意思了。

   我们可以发现，如果点 i 为割点，显然去掉这个点之后整个图会变成几个联通块，如下图：

   ![](https://s2.ax1x.com/2019/09/08/n3MWFO.png)

   这种情况我们也很好发现，把联通块的大小两两相乘可得答案。

   记第 i 个联通块为$s_i$

   但是把联通块大小两两相乘的复杂度为 $O(n^2)$ 不能接受，我们可以在 dfs 时把搜索树子树大小算出来，记为 $siz[i]$

   最后的答案即为： 

   $(n - 1 - \sum_{i=1}^{t}siz[s_k])*(1+\sum_{i=1}^{t}siz[s_k])$ 

## 代码：

```cp
#include <cstdio>
#include <cctype>
#include <algorithm>
using namespace std;
const int N = 100010;
const int M = 500010<<1;
inline int read() {
	int x = 0,f = 1;char v = getchar();
	while (!isdigit(v)) {if (v =='-') f = -1;v = getchar();}
	while (isdigit(v)) {x = x * 10 + v - 48;v = getchar();}
	return x * f;
}
int nxt[M],hd[N],to[M],tot = 1,cnt,dfn[N],low[N],siz[N],n,m;
long long ans[N];
bool cut[N];

inline void adde(int u,int v) {
	to[++tot] = v;nxt[tot] = hd[u];hd[u] = tot;
}

inline void addedge(int u,int v) {
	adde(u,v);adde(v,u);
}

void tarjan(int x) {
	dfn[x] = low[x] = ++cnt;
	siz[x] = 1;
	int flag = 0,sum = 0;
	for (int i = hd[x];i;i = nxt[i]) {
		int v = to[i];
		if (!dfn[v]) {
			tarjan(v);
			low[x] = min(low[x],low[v]);
			siz[x] += siz[v];
			if (low[v] >= dfn[x]) {
				flag++;
				ans[x] += (long long)siz[v]*(n - siz[v]);
				sum += siz[v];
				if (x != 1 || flag > 1) {
					cut[x] = 1;
				}
			}
		}	
		else {
			low[x] = min(low[x],dfn[v]);
		}
	}
	if (cut[x]) {
		ans[x] += (long long)(n - sum - 1) * (sum + 1) + (n - 1);
	}
	else {
		ans[x] = 2*(n-1);
	}
}

int main() {
	n = read(),m = read();
	for (int i = 1;i <= m;++i) {
		int x = read(),y = read();
		if (x == y) {
			continue;
		}
		addedge(x,y);
		
	}
	tarjan(1);
	for (int i = 1;i <= n;++i) {
		printf("%lld\n",ans[i]);
	}
	return 0;
}
```

## 参考：

部分思路来自于lyd的《算法竞赛进阶指南》

