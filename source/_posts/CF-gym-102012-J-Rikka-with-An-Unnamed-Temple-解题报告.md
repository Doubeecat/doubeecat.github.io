title: CF gym 102012 J Rikka with An Unnamed Temple 解题报告
categories: 解题报告
tags: [拓扑排序,DP,线段树,数据结构,毒瘤]
date: 2021-03-05 18:39:26
---

> [CF gym 102012 J](https://codeforces.ml/gym/102012/problem/J)
> 
> 给出一张有向无环图，每个点上存储了一个具有特定重量和价值的宝石，经过一个点时必须拿取上面的宝石。
>
> 对于每个点求出：
>
> 禁止经过这个点时，从起点走到终点，且路径上所有宝石重量之和除以 $m$ 的余数为 $k$ 时，所能得到的最大收益。
>
> $n,m \leq 200000$

<!--more-->

Sol：

神仙题，本来要睡觉了，结果看了一眼这个题，所以现在快三点了。

首先注意到这是一个 DAG ，那么我们可以判断这个东西必定和拓扑序沾点。

接下来我们考虑一下一个点被禁止经过了之后发生了什么。一个点被禁止经过后对和这个点直接相连的点有影响，而我们的目的就是要找到和这个点直接相连的点中最大的 $c_y$。

这个东西显然是一个线段树维护，然后我们可以很自然想到对整个图按照拓扑序进行 DP。对原图进行一次 DP，再对反图进行一次 DP，可以求出 $f1_x$ 和 $f2_x$ 即从 $1 \rightarrow x$ 和 $x \rightarrow n$ 的最大价值，对于每个点算的时候就直接按照上面用线段树维护即可。

再注意到这题是模数相关的，我们 DP 的时候枚举一下最后的余数就解决了。

说是这么说，但是代码很难写。

Code：

血 泪 史

```cpp
#include <algorithm>
#include <iostream>
#include <cstring>
#include <cstdio>
#include <cctype>
#include <vector>
#include <cmath>
#include <queue>
using namespace std;
#define int long long
#define ll long long
#define ri register int
#define pii pair<ll,ll>
#define mp make_pair

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

const int N = 1e5 + 10;
const int W = 110;
const ll mod = 1000000007;

vector <int> G1[N],G2[N];

int n,m,deg[N],topo[N],cnt,w[N],c[N],k,t,qwq[N];
pii f1[N][W],f2[N][W],ans[N];

void merge(pii& a,pii b) {
    if (a.first < b.first) {
        a = b;
        return ;
    } else if (a.first == b.first) {
        a = mp(a.first,(a.second + b.second) % mod);
        return ;
    }
}

void toposort() {
    for (int i = 1;i <= n;++i) if (!deg[i]) qwq[++cnt] = i;
    for (int i = 1;i <= n;++i) {
        for (auto y:G1[qwq[i]]) {
            if (!--deg[y]) {
                qwq[++cnt] = y;
            }
        }
    }
    for (int i = 1;i <= n;++i) topo[qwq[i]] = i;

}

struct node {
    int l,r;
    pii data;
}tree[N<<2];

void build(int p,int l,int r) {
    tree[p].l = l,tree[p].r = r,tree[p].data = mp(-1,0);
    if (l == r) {
        ans[l] = mp(-1,0);
        return ;
    }
    int mid = (l + r) >> 1;
    build(p<<1,l,mid);
    build(p<<1|1,mid+1,r);
}

void pushdown(int p) {
    if (tree[p].data.first == -1) return ;
    merge(tree[p<<1].data,tree[p].data);merge(tree[p<<1|1].data,tree[p].data);
    tree[p].data = mp(-1,0);
}

void modify(int p,int l,int r,pii v) {
    if (tree[p].l >= l && tree[p].r <= r) {
        merge(tree[p].data,v);
        return ;
    }
    int mid = (tree[p].l + tree[p].r) >> 1;
    if (l <= mid) modify(p<<1,l,r,v);
    if (r > mid) modify(p<<1|1,l,r,v);
}

void calc(int p,int l,int r) {
    if (l == r) {
        ans[l] = tree[p].data;
        return ;
    }
    pushdown(p);
    int mid = (tree[p].l + tree[p].r) >> 1;
    calc(p<<1,l,mid);
    calc(p<<1|1,mid+1,r);
}
void solve() {
    memset(tree,0,sizeof tree);
    memset(ans,0,sizeof ans);
    cnt = 0;
    for (int i = 1;i <= n;++i) G1[i].clear(),G2[i].clear(),deg[i] = 0,qwq[i] = 0,topo[i] = 0;
    read(n,m);
    for (int i = 1;i <= n;++i) read(w[i],c[i]);
    for (int i = 1;i <= m;++i) {
        int x,y;read(x,y);
        G1[x].push_back(y);G2[y].push_back(x);
        ++deg[y];
    }
    toposort();
    read(k,t);
    for (int i = 1;i <= n;++i) {
        w[i] %= k;  
        for (int j = 0;j != k;++j) {
            f1[i][j] = f2[i][j] = mp(-1,0);
        }
    }
    //f1 is from the positive direction , so the first point is n
    //f2 is from the opposite direction, so the first point is 1    
    f1[n][w[n]] = mp(c[n],1),f2[1][w[1]] = mp(c[1],1);  
    for (int i = 1;i <= n;++i) {
        int x = qwq[i];
        for (auto y:G2[x]) {
            for (int j = 0;j != k;++j) {
                if (f2[y][j].first != -1) {
                    merge(f2[x][(j + w[x]) % k],mp(f2[y][j].first + c[x],f2[y][j].second));
                    //考虑一下转移，如果当前点为x然后模数为j，那么实际上走到y这个点就是把c[x]的代价给合并过去
                }
            }
        }
    }
    for (int i = n;i >= 1;--i) {//注意美剧顺序！
        int x = qwq[i];
        for (auto y:G1[x]) {
            for (int j = 0;j != k;++j) {
                if (f1[y][j].first != -1) {
                    merge(f1[x][(j + w[x]) % k],mp(f1[y][j].first + c[x],f1[y][j].second));
                    //考虑一下转移，如果当前点为x然后模数为j，那么实际上走到y这个点就是把c[x]的代价给合并过去
                }
            }
        }
    }

    build(1,1,n);

    for (int i = 1;i <= n;++i) {
        for (auto j:G1[i]) {
            int x = topo[i]+1,y = topo[j] - 1;//正好是拓扑序上前后两点
            if (x > y) continue;
            pii tmp = mp(-1,0);
            for (int s = 0;s < k;++s) {
                pii v1 = f2[i][s],v2 = f1[j][(t-s + k)%k];//排除掉x这点.
                if (v1.first == -1 || v2.first == -1) continue;
                //printf("%d %d\n",j,(t-s+k)%k);
                //printf("%lld %lld %lld %lld %lld\n",v1.first,v1.second,v2.first,v2.second,(v1.second * v2.second) % mod);
                merge(tmp,mp(v1.first + v2.first,(ll)((v1.second * v2.second) + mod) % mod));
            }
            if (tmp.first != -1) {
                modify(1,x,y,tmp);
            }
        }
    }

    if (topo[1] != 1) modify(1,1,topo[1]-1,f1[1][t]);
    if (topo[n] != n) modify(1,topo[n]+1,n,f2[n][t]);

    calc(1,1,n);

    for (int i = 1;i <= n;++i) {
        int x = topo[i];
        if (ans[x].first != -1) printf("%lld %lld\n",ans[x].first,ans[x].second);
        else puts("-1");
    }
}

signed main() {
	int T;read(T);
    while (T--) solve();
	return 0;
}
```