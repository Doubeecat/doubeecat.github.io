---
title: CF1706E Qpwoeirut and Vertices 解题报告
categories: 解题报告
tags: [Kruskal 重构树,RMQ,LCA]
date: 2022-07-20 02:51:00
---
# E. Qpwoeirut and Vertices

> 给定一张 $n$ 个点 $m$ 条边的无向连通图，有 $q$ 个询问，每次询问给定两个数 $l,r$，请你给出最小的 $k$ 使得：
>
> 仅使用前 $k$ 条边就能使所有点对 $(a,b)$ 联通，其中 $a,b$ 满足 $l \leq a \leq b \leq r$。
>
> $n \leq 10^5,m,q \leq 2\times 10^5$

<!--more-->

## 分析

这个题其实和 [AGC002D](https://atcoder.jp/contests/agc002/tasks/agc002_d) 有着一定的相似之处，这类在图上求一些最值的题比较常见的套路要么是线段树分治，要么是 Kruskal 重构树。

这题线段树分治显然是没有什么前途的，所以我们考虑 Kruskal 重构树。由于这题求的是按照边的顺序，所以我们把边的权值赋为边的序号，排序后建出 Kruskal 重构树。

那么我们现在的问题就变成了，对于在 $[l,r]$ 内的点，我们需要找到他们的 LCA 来保证是能联通的（这个就是 Kruskal 重构树的基本性质了，不再赘述）。对于求多个点 LCA 的问题，我们可以给出结论：**树上多个点的 LCA，就是 DFS 序最小的和 DFS 序最大的这两个点的 LCA。**详细的证明可以戳 [如何在 DAG 中找多个点的 LCA ? - 阮行止的回答 - 知乎](https://www.zhihu.com/question/46440863/answer/165741734)

所以我们就在 $\mathcal{O}((m+q)\log n)$ 的时间内解决了本题。

## 代码

考场代码 最后 2 min 过的，所以略粗糙。

```cpp
#pragma GCC optimize(2)
#include <algorithm>
#include <iostream>
#include <cstring>
#include <cstdio>
#include <cctype>
#include <vector>
#include <cmath>
using namespace std;
#define pii pair<int,int>
#define mp make_pair
#define ll long long

char buf[1 << 20], *p1, *p2;
#define getchar() (p1 == p2 && (p2 = (p1 = buf) + fread(buf, 1, 1 << 20, stdin), p1 == p2)?EOF: *p1++)
template <typename T> inline void read(T &t) {
	int v = getchar();T f = 1;t = 0;
	while (!isdigit(v)) {if (v == '-')f = -1;v = getchar();}
	while (isdigit(v)) {t = t * 10 + v - 48;v = getchar();}
	t *= f;
}
template <typename T,typename... Args> inline void read(T &t,Args&... args) {
	read(t);read(args...);
}
const int N = 4e5 + 10;
const int K = 26;

int n,m,q,k,s,dis[N],F[N],poi[N],val[N],tot,fat[N][K],dep[N],dfn[N],rk[N],cnt,siz[N];
bool vis[N];
vector <int> G[N];
vector <pii> G2[N];

struct edg {
    int u,v,w;
    friend inline bool operator < (const edg &A,const edg &B) {
        return A.w < B.w;
    }
}edge[N];

int find(int x) {return x == F[x] ? x : F[x] = find(F[x]);}

void clear() {
    tot = n;cnt = 0;
    for (int i = 1;i <= (n << 1);++i) G[i].clear(),F[i] = i,dfn[i] = rk[i] = 0;
    //memset(fat,0,sizeof fat);
}

void Kruskal() {
    //poi[0] = 0x3f3f3f3f;
    for (int i = 1;i <= m;++i) {
        int u = edge[i].u,v = edge[i].v,w = edge[i].w;
        //printf("%d %d %d %d\n",u,v,find(u),find(v));
        u = find(u),v = find(v);
        if (u != v) {
            ++tot;
            poi[tot] = w;
            //poi[u] = poi[v] = w;
            F[u] = F[v] = tot;
            G[tot].push_back(u);
            G[u].push_back(tot);
            G[tot].push_back(v);
            G[v].push_back(tot);
        }
    }
}

void dfs(int x,int fa) {
    fat[x][0] = fa;
    dep[x] = dep[fa] + 1;
    dfn[x] = ++cnt;rk[cnt] = x;
    //printf("%d %d\n",fa,x);
    for (int j = 1;j <= 20;++j) {
        fat[x][j] = fat[fat[x][j-1]][j-1];
    }
    for (auto y : G[x]) {
        if (y != fa) {
            dfs(y,x);
        }
    }
}

int LCA(int x,int y) {
    if (dep[x] > dep[y]) swap(x,y);
    for (int i = 20;i >= 0;--i) {
        if (dep[fat[y][i]] >= dep[x]) {
            y = fat[y][i];
        }
    }
    if (x == y) return x;
    for (int i = 20;i >= 0;--i) {
        if (fat[x][i] != fat[y][i]) {
            x = fat[x][i];
            y = fat[y][i];
        }
    }
    return fat[x][0];
}

int st1[K][N],st2[K][N];

void init() {
    for (int j = 0;j <= 20;++j) {
        for (int i = 1;i + (1 << j) - 1 <= n;++i) {
            if (!j) st1[j][i] = st2[j][i] = dfn[i];
            else {
                st1[j][i] = min(st1[j-1][i],st1[j-1][i + (1 <<j-1)]);
                st2[j][i] = max(st2[j-1][i],st2[j-1][i + (1 <<j-1)]);
            }
        }
    }
}

int query1(int l,int r) {
    int now = log(r - l + 1) / log(2);
    return min(st1[now][l],st1[now][r - (1 << now) + 1]);
}

int query2(int l,int r) {
    int now = log(r - l + 1) / log(2);
    return max(st2[now][l],st2[now][r - (1 << now) + 1]);
}


void solve() {
    int q;
    read(n,m,q);
    clear();
    for (int i = 1;i <= m;++i) {
        read(edge[i].u,edge[i].v);
        edge[i].w = i;
    }
    sort(edge + 1,edge + 1 + m);
    Kruskal();
    dfs(tot,0);init();
    //for (int i = 1;i <= n;++i) printf("i:%d ",dfn[i]);
    //puts("");
    for (int i = 1;i <= q;++i) {
        int l,r;read(l,r);
        if (l == r) printf("%d ",0);
        else {
            int u = rk[query1(l,r)],v = rk[query2(l,r)];
            //printf("u,v:%d %d l:%d\n",u,v,query1(l,r));
            printf("%d ",poi[LCA(u,v)]);
        }
    }
    puts("");
}

signed main() {
    int T;read(T);
    while (T--) solve();
	return 0;
}

```

