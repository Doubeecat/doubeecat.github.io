---
title: THUPC2022初赛 A.最小公倍树 解题报告
categories: 解题报告
tags: [数学,Kruskal]
date: 2022-03-12 12:06:00
---
> [THUPC2022 A.最小公倍树](https://thupc2022.thusaac.com/#!/contest/2/problem/0)
>
> 给定一个点编号在 $[L,R]$ 范围内的完全图，边 $(u,v)$ 的权值为 $\mathrm{lcm}(u, v)$，请你求出这张图的最小生成树权值和。
>
> $L,R \leq 10^6,R - L \leq 10^5$

<!--more-->

## 解题思路：

Kruskal 优化建图。

观察到如果直接建图的时间复杂度是 $\mathcal{O}((R-L)^2	)$ 的，无法接受。但是发现在这个过程中，有很多边实际上是可以省略的，接下来来发掘一下这些边的性质。

首先我们有 
$$
\mathrm{lcm}(u, v) = \frac{u\times v}{\mathrm{gcd}(u,v)}
$$
这实际上启发了我们从 $\mathrm{gcd}(u,v)$ 的角度进行思考，我们发现，如果将两个有共同因子的点连边，这条边的权值是较小的。观察到 $L,R \leq 10^6$，这启发我们去枚举这个因子。同时我们发现，如果对于一个因子 $x$ 来说，在 $[L,R]$ 里的数都满足 $kx,(k+1)x\dots (k+p)x$ 这样的形式，显然对于 $(k+1)x$ 及后面的数，向 $kx$ 连边是最优秀的，这样我们对于一个因子最多建边数量为 $\lfloor\frac{R-L}{x}\rfloor$ 的。

而我们可以枚举因子，因子的范围是 $[2,R]$ ，最后我们建边的数量就应该是 $\lfloor\frac{R-L}{2}\rfloor + \lfloor\frac{R-L}{3}\rfloor  + \dots + \lfloor\frac{R-L}{R}\rfloor$ 根据调和级数，这个东西的边数是 $\mathcal{O}((R-L) \log R)$ 的，实际上这个上界很松，具体数据会更小，极限数据大概是 $10^5$ 左右的，建出边后跑 Kruskal 就过了。

时间复杂度  $\mathcal{O}(((R-L) \log R) \log ((R-L) \log R))$ 。

## 代码：

```cpp
const int N = 1e6 + 10;

int l,r,f[N],buc[N];
bool vis[N];

ll gcd(ll a,ll b) {
    return !b ? a : gcd(b,a%b);
}

ll lcm(ll a,ll b) {
    return a / gcd(a,b) * b;
}

int find(int x) {return f[x] == x ? x : f[x] = find(f[x]);}

struct node {
    int l,r;
    ll w;
    friend inline bool operator < (const node &a,const node &b) {
        return a.w < b.w;
    }
};

vector <node> edge;

void Kruskal() {
    sort(edge.begin(),edge.end());
    int cnt = 0;
    ll ans = 0;
    for (auto e : edge) {
        int x = e.l,y = e.r;ll w = e.w;
        if (find(x) != find(y)) {
            //printf("%d %d %lld\n",x,y,w);
            f[max(find(x),find(y))] = min(find(x),find(y));
            ans += w;
        }
    }
    printf("%lld\n",ans);
}

signed main() {
    read(l,r);
    ll ans = 0;
    for (int i = l;i <= r;++i) f[i] = i,buc[i] = 1;
    for (int i = 2;i <= r;++i) {
        //if (vis[i]) continue;
        int cnt = 0,fis = 0;
        
        for (int j = i;j <= r;j += i) {
            if (buc[j] && !fis) fis = j;
            if (buc[j]) edge.push_back((node){fis,j,lcm(fis,j)});
            vis[j] = 1;
        }

        if (i >= l) edge.push_back((node){fis,l,lcm(fis,l)});
    }
    Kruskal();
	return 0;
}

```



