title: Luogu P2169 正则表达式 解题报告
categories: 解题报告
tags: [图论,Tarjan,缩点]
date: 2019-10-11 15:16:00
---
> [P2169 正则表达式](<https://www.luogu.org/problem/P2169>)
> 在Internet网络中的每台电脑并不是直接一对一连通的，而是某些电脑之间存在单向的网络连接，也就是说存在A到B的连接不一定存在B到A的连接，并且有些连接传输速度很快，有些则很慢，所以不同连接传输所花的时间是有大有小的。另外，如果存在A到B的连接的同时也存在B到A的连接的话，那么A和B实际上处于同一局域网内，可以通过本地传输，这样花费的传输时间为0。

> 现在小Z告诉你整个网络的构成情况，他希望知道从他的电脑（编号为1），到小X的电脑（编号为n）所需要的最短传输时间。

> 对于100%的数据，$1\leq n\leq200000, 1\leq m\leq 1000000$


<!--more-->
## 解题思路：

Tarjan 缩点 + SPFA/DP 

首先观察数据范围，显然这么大的 $n,m$ ，直接跑 $O(nm)$ 的 SPFA 会炸掉。

观察题面，发现如果有 x 和 y 构成一个环（处在同一个 SCC 中），那么他们之间的边长度为 0 

这启发我们可以用缩点后再来跑 SPFA ，但是这样的复杂度是不够严谨的，最坏情况下仍旧是 $O(nm)$ 的，只是由于这题水所以可以过。

## 代码：

SPFA 版本 

```cpp
#include <cstdio>
#include <cctype>
#include <cmath>
#include <stack>
#include <cstring>
#include <queue>
using namespace std;
inline int read() {
	char v = getchar();int x = 0,f = 1;
	while (!isdigit(v)) {if (v == '-')f = -1;v = getchar();}
	while (isdigit(v)) {x = x * 10 + v - 48;v = getchar();}
	return x * f;
}
const int N = 10010;
const int M = 5000010;

int to[M],hd[N],nxt[M],tot,edg[M];
inline void add(int u,int v,int w) {to[++tot] = v;edg[tot] = w;nxt[tot] = hd[u];hd[u] = tot;}

inline int min(int x,int y) {return x<y?x:y;}
inline int max(int x,int y) {return x>y?x:y;}

using std::stack;

int dfn[N],low[N],ins[N],cnt,num,c[N],n,m,p[N],f[N];
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
		c[x] = ++num;int y;
		do {
			y = s.top(),s.pop();
			ins[y] = 0;c[y] = num;
		}while (x != y);
	}
}

int vis[N],dis[N];

void SPFA(int s) {
	memset(dis,0x3f,sizeof(dis));
	queue <int> q;
	vis[s] = 1;dis[s] = 0;
	q.push(s);
	while (!q.empty()) {
		int x = q.front();q.pop();
		vis[x] = 0;
		for (int i = hd[x];i;i = nxt[i]) {
			int y = to[i],w = edg[i];
			if (c[x] == c[y]) w = 0;
			if (dis[y] > dis[x] + w) {
				dis[y] = dis[x] + w;
				if (!vis[y]) {
					q.push(y);
					vis[y] = 1;
				}
			}
		}
	}
}

int main() {
	n = read(),m = read();
	for (int i = 1;i <= m;++i) {
		int x = read(),y = read(),w = read();add(x,y,w);
	}
	for (int i = 1;i <= n;++i) if (!dfn[i]) tarjan(i);
	SPFA(1);
	printf("%d",dis[n]);
	return 0;
}

```