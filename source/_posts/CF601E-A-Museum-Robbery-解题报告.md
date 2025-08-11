title: CF601E A Museum Robbery 解题报告
categories: 解题报告
tags: [背包,分治,线段树分治]
date: 2022-03-02 12:40:00
---
> [CF601E A Museum Robbery](http://codeforces.com/problemset/problem/601/E)
>
> 最初给定 $n$ 个物品以及背包容量 $k$，有 $q$ 次操作，操作有三种：
>
> - `1 v w` 在背包里添加一个体积为 $v$ 价值为 $w$ 的物品
> - `2 x` 删除编号为 $x$ 的物品
> - `3` 查询背包总和，以 $\sum\limits_{m=1}^{k}{s(m)*p^{m-1}\ \bmod\ q}$ 的形式输出
>
> $n \leq 5000,k \leq 1000,q \leq 30000$ ，保证操作 $1$ 的个数不超过 $10000$，且至少有一个操作 $3$。

<!--more-->

## 解题思路：

首先我们考虑，在背包里加入一个物品是 $\mathcal{O}(n)$ 的，但是删除一个物品并不好处理，只想到了重做一遍背包的方法，但是这样是 $\mathcal{O}(n^2)$ 的，显然会寄。

接下来观察题目的前两个操作，发现如果我们把最开始的物品看作在时间点 $0$，那么实际上所有物品可以考虑成在一个时间段上出现，这个就很符合**线段树分治**的套路了。

确定了算法之后，就开始考虑怎么写。首先我们记操作 $3$ 的个数有 $cnt$ 个，首先线段树建在 $[1,cnt]$ 上，每个线段树节点上考虑开一个 `vector` 来保存这个节点代表的区间存在的物品。我们发现这里实际上不好合并，所以我们采用**标记永久化**的方法。

在分治过程中，我们每访问到一个节点考虑一个额外的 $g$ 数组作背包用，将 $f$ 拷贝到 $g$ 后对于 $g$ 进行新增物品的背包操作。而在分治之后直接下传 $g$ 数组，于是我们可以写出一个伪代码框架了：

```cpp
void solve(int p,ll *f) {
    ll g[N];
    for (int i = 0;i <= sto;++i) g[i] = f[i];
    for (auto now : tree[p].vec) {
        Add(g,now);//将物品 now 添加到 g 中。
    }
    if (tree[p].l == tree[p].r) {
        Output();
        return ;
    } else {
        //注意此处下传是 g 数组，已经添加过物品了。
        solve(p << 1,g);
        solve(p << 1 | 1,g);
    }
}
```

时间复杂度 $\mathcal{O}((n+q)\log (n+q) + q)$。注意数组如果开的较大不要使用 `memset`。

## 代码：

```cpp
const int N = 1e5 + 10;
const ll base = 1e7 + 19;
const ll mod = 1e9 + 7;

struct item {
    int x,y,v,w;
}E[N];

struct node {
    int l,r;
    vector <item> vec;
}tree[N<<2];

ll qwq[N];
int tot,n,q,sto;

void build(int p,int l,int r) {
    tree[p].l = l,tree[p].r = r;
    if (l == r) {
        return ;
    }
    int mid = (l + r) >> 1;
    build(p << 1,l,mid);
    build(p << 1 | 1,mid + 1,r);
}

void change(int p,int x,int y,int num) {
    if (tree[p].l >= x && tree[p].r <= y) {
        tree[p].vec.push_back(E[num]);
        return ;
    }
    int mid = (tree[p].l + tree[p].r) >> 1;
    if (x <= mid) change(p << 1,x,y,num);
    if (y > mid) change(p << 1 | 1,x,y,num);
}

void solve(int p,ll *f) {
    ll g[N];
    for (int i = 0;i <= sto;++i) g[i] = f[i];
    for (auto now : tree[p].vec) {
        for (int j = sto;j >= now.v;--j) {
            g[j] = max(g[j],g[j - now.v] + now.w); 
        }
    }
    if (tree[p].l == tree[p].r) {
        ll ans = 0;
        for (int i = sto;i >= 1;--i) ans = (ans * base + g[i]) % mod;
        printf("%lld\n",ans);
    } else {
        solve(p << 1,g);
        solve(p << 1 | 1,g);
    }
}


signed main() {
    read(n,sto);
    for (int i = 1;i <= n;++i) {
        int v,w;read(w,v);
        E[++tot] = {1,-1,v,w};
    }
    int tt = 1;
    read(q);
    for (int i = 1;i <= q;++i) {
        int opt;read(opt);
        if (opt == 1) {
            int v,w;read(w,v);
            E[++tot] = {tt,-1,v,w};
        }
        if (opt == 2) {
            int x;read(x);
            E[x].y = tt - 1;
        }
        if (opt == 3) {
            ++tt;
        }
    }
    --tt;
    build(1,1,tt);
    for (int i = 1;i <= tot;++i) {
        if (E[i].y == -1) E[i].y = tt;
        if (E[i].x <= E[i].y) {
            change(1,E[i].x,E[i].y,i);
        }
    }
    solve(1,qwq);
	return 0;
}
```