title: Luogu P2700 逐个击破 解题报告
categories: 解题报告
tags: [图论,并查集,思维,树上问题,Kruskal]
date: 2020-09-11 14:02:26
---

> [P2700 逐个击破](https://www.luogu.com.cn/problem/P2700)
> 
> 一棵树上有 $k$ 个关键点，要求用最小的代价把这些点划分到不同连通块中。
> 
> $2 \leq k \leq n \leq 10^5$

<!--more-->

## 解题思路

很巧妙的思维题。

首先一个很暴力的思路是，对于每个关键点求出与其他关键点的路径，然后进行树上差分，但是这样的复杂度是 $O(k^2 \log n)$ 的，而且非常难写。

我们可以从题面中观察出一些性质：

1. 对于 $k$ 个点，最多只需要 $k-1$ 条边就可以把他们划分开。

    证明十分显然，手玩一下就出来了。

2. 对于每个点所在的连通块来说，关键点与其他非关键点的连边并不影响答案

    因为每个连通块我们实际上只需要考虑这个关键点与其他关键点的连边，这个关键点与非关键点的连边并不重要。

这些性质看上去没什么用，似乎没办法了？我们还可以反向思考一下

划分到不同连通块，实际上相当于**把一个关键点与其他非关键点连通，并且不与其他关键点联通**，也就把删边转化为了加边。

结合性质 2，我们可以考虑一下贪心，把所有最大的并且不会导致两个关键点联通的边加上。

这让我们联想到了 Kruskal，于是，我们可以执行一个类似 Kruskal 的过程：

首先对所有边进行从大到小的排序，然后从大到小加边，每次加边的时候判断是否加上这条边就会有两关键点联通。如果可以加上，就从总边权减去当前边的边权（因为我们的目的是求让连通块断开的最小权值）这个过程用并查集可以很方便的维护。

于是我们就解决了这个问题，时间复杂度 $O(n \log n)$，瓶颈在于排序。

## 代码

```cpp

ll ans,k,p[N],n,f[N];
bool vis[N];

struct edge {
    int u,v,w;
    friend inline bool operator < (const edge & a,const edge &b) {
        return a.w > b.w;
    }
}edg[N];

int find(int x) {return f[x] == x ? x : f[x] = find(f[x]);}
bool merge(int x,int y) {
    int fx = find(x),fy = find(y);
    if (!vis[fx] || !vis[fy]) {//合并的时候判断两边是否全部都有点，如果有就不连了
        f[fx] = fy;
        if (vis[fy] == 1) vis[fx] = 1;
        else if (vis[fx] == 1) vis[fy] = 1;
        //把连通块中关系继承
        return 1;
    }
    return 0;
}

signed main() {
	read(n,k);
    for (int i = 1;i <= k;++i) {
        int x;read(x);vis[x] = 1;
    }
    for (int i = 1;i <= n;++i) f[i] = i;
    for (int i = 1;i < n;++i) {
        read(edg[i].u,edg[i].v,edg[i].w);ans += edg[i].w;
    }
    sort(edg+1,edg+n);
    for (int i = 1;i < n;++i) {
        edge e = edg[i];
        int u = e.u,v = e.v;
        if (merge(u,v)) ans -= e.w;
        //如果有边连就扣去
    }
    printf("%lld",ans);
	return 0;
}
```