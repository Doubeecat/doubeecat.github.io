title: CF gym 102201F Fruit tree 解题报告
categories: 解题报告
tags: [树上问题,数据结构,树状数组,DFS序]
date: 2021-03-02 13:13:51
---

> [CF gym 102201F Fruit tree](https://codeforces.com/gym/102201/problem/F)
> 
> 有一棵 $n$ 个节点的树，每个节点上有一个颜色，有 $q$ 次询问，每次询问给定两个点 $u,v$，要求你求出是否有一种颜色在 $u,v$ 的简单路径上出现超过一半次数。
> 
> $n,q \leq 2.5 \times 10^5$


<!--more-->

Solution:

首先先考虑一下序列上的版本怎么做：

> 给你一个序列，每个位置有一个元素，多次询问，每次求出 $[l,r]$ 中是否有出现次数大于 $\frac{r-l+1}{2}$ 的颜色。

这个问题可以使用莫队在 $O(n\sqrt n)$ 的时间内解决，也可以通过主席树在 $O(n\log n)$ 的时间内解决。但是我们这里引入一种更加简单，并且时间复杂度仍然是 $O(n \log n)$的算法。

前置知识：[摩尔投票法](https://blog.csdn.net/happyeveryday62/article/details/104136295)

在你知道了摩尔投票法之后，我们来理解一下以下推论：

将二元组 $(x_1,c_1),(x_2,c_2)$ (保证 $c_1 > c_2$) 合并的做法为：

若 $x_1 \not = {x_2}$，则合并后的二元组为 $(x_1,c_1-c_2)$

若 $x_1 = {x_2}$，则合并后的二元组为 $(x_1,c_1+c_2)$

证明十分显然，代码这么写：

```cpp
pii merge(pii a,pii b) {
    if (a.first != b.first) {
        if (a.second >= b.second) {
            return make_pair(a.first,a.second - b.second);
        } else {
            return make_pair(b.first,b.second - a.second);
        }
    } else {
        return make_pair(a.first,a.second + b.second);
    }
}
```

基于这个，我们可以使用倍增在 $O(n \log n)$ 的时间内预处理出第 $i$ 个元素到第 $i + 2 ^ j$ 个元素的**代表颜色**，然后对于每个区间，我们再用二进制拼凑法算出每个区间的 **代表颜色**。

那么接下来的问题是，我们并不能保证每种代表颜色就一定是正确（即出现次数大于一半）的颜色。所以我们将每种颜色的点和询问单独提出来，然后按颜色处理。

对于每种颜色，我们先在树状数组上将该种颜色的点的前缀和全部 +1，然后直接查询对应区间内的颜色是否满足条件即可。时间复杂度 $O(n\log n)$

那么这个问题实际上就是把序列上问题上树了，把倍增换成树上倍增，在 DFS 序上建树状数组，然后按照序列上方法处理即可。

Code:

```cpp
#include <algorithm>
#include <iostream>
#include <cstring>
#include <cstdio>
#include <cctype>
#include <vector>
#include <cmath>
using namespace std;
#define ll long long
#define ri register int
#define pii pair<int,int>

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

const int N = 250001;
const int K = 21;

vector <int> G[N],c[N],c1[N];

int cnt,in[N],out[N],t,f[N][K],dep[N],a[N],n,q,st[N],ed[N],lca[N],ans[N],co[N],len[N];
pii col[N][K];

pii merge(pii a,pii b) {
    if (a.first != b.first) {
        if (a.second >= b.second) {
            return make_pair(a.first,a.second - b.second);
        } else {
            return make_pair(b.first,b.second - a.second);
        }
    } else {
        return make_pair(a.first,a.second + b.second);
    }
}

struct BIT{
    int qwq[N*2];

    int lowbit(int x) {return (x & (-x));}

    void modify(int x,int v) {
        for (;x <= cnt;x += lowbit(x)) qwq[x] += v;
    }

    int query(int x) {
        int ans = 0;
        for (;x;x -= lowbit(x)) ans += qwq[x];
        return ans;
    }
}T;

void dfs(int x,int fa) {
    col[x][0] = make_pair(a[x],1);
    in[x] = ++cnt;
    for (int i = 1;i <= t;++i) {
        f[x][i] = f[f[x][i-1]][i-1];
    }
    for (int i = 1;i <= t;++i) {
        col[x][i] = merge(col[x][i-1],col[f[x][i-1]][i-1]);
    }
    for (auto y:G[x]) {
        if (y != fa) {
            f[y][0] = x;
            dep[y] = dep[x] + 1;
            dfs(y,x);
        }
    }
    out[x] = ++cnt;
}

int LCA(int x,int y) {
    if (dep[x] < dep[y]) swap(x,y);
    for (int d = dep[x] - dep[y],i = 0;d;d >>= 1,++i) if (d & 1) x = f[x][i];
    if (x == y) return x;
    for (int i = t;i >= 0;--i) {
        if (f[x][i] != f[y][i]) {
            x = f[x][i],y = f[y][i];
        }
    }
    return f[x][0];
}

int work(int x,int y) {
    pii now = make_pair(1,0);
    if (dep[x] < dep[y]) swap(x,y);
    for (int d = dep[x] - dep[y],i = 0;d;d >>= 1,++i) if (d & 1) now = merge(now,col[x][i]),x = f[x][i];
    if (x == y) return merge(now,col[x][0]).first;
    for (int i = t;i >= 0;--i) {
        if (f[x][i] != f[y][i]) {
            now = merge(now,col[x][i]),now = merge(now,col[y][i]);
            x = f[x][i],y = f[y][i];
        }
    }
    now = merge(now,col[x][0]);
    now = merge(now,col[y][1]);
    return now.first;
}

signed main() {
    read(n,q);
    t = 20;
    for (int i = 1;i <= n;++i) {
        read(a[i]);c[a[i]].push_back(i);
    }
    for (int i = 1;i < n;++i) {
        int x,y;read(x,y);G[x].push_back(y),G[y].push_back(x);
    }
    dfs(1,0);

    for (int i = 1;i <= q;++i) {
        ans[i] = -1;
        read(st[i],ed[i]);
        lca[i] = LCA(st[i],ed[i]);
        co[i] = work(st[i],ed[i]);
        c1[co[i]].push_back(i);
        len[i] = dep[st[i]] + dep[ed[i]] - 2 * dep[lca[i]] + 1;

    }
    for (int i = 1;i <= n;++i) {
        for (auto y:c[i]) {
            T.modify(in[y],1),T.modify(out[y],-1);
        }
        for (auto y:c1[i]) {
            //printf("%d\n",y);
            int u = st[y],v = ed[y],p = lca[y];
            int tmp = T.query(in[u]) + T.query(in[v]) - 2 * T.query(in[p]-1) - (a[p] == i);
            //printf("%d %d %d %d %d %d\n",i,y,tmp,len[y],u,v);
            if (tmp * 2 > len[y]) ans[y] = i;
        }
        for (auto y:c[i]) {
            T.modify(in[y],-1),T.modify(out[y],1);
        }
    }
    for (int i = 1;i <= q;++i) printf("%d\n",ans[i]);
	return 0;
}
```