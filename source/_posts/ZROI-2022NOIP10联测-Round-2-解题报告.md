---
title: ZROI 2022NOIP10联测 Round 2 解题报告
categories: 解题报告
tags: [树上问题,拉格朗日插值]
date: 2022-09-05 10:28:00
---
100 + 60 + 40 + 30,rk 17.

昨天的 CSP 由于太逆天不补了。

<!--more-->

# A

这个题最重要的是抓住一个性质：保证 $a_i$ 和所有操作 1 的 $x$ 总共最多不超过 $10$ 种数。

所以我们考虑可以预处理出 $a_i$ 的每个子集是否能凑出来对应的 $x$，这可以使用完全背包在 $\mathcal{O}(2^{10}\times m)$ 的时间内完成。

那么我们考虑操作 1 和 2 怎么维护，我们可以维护一个线段树。每个节点上带有一个 $val$ 表示当前节点维护的区间内子集的情况。然后对于操作 1 我们就直接区间推平，操作 2 查询出来后先存到桶里然后对于每个子集做一遍背包就做完了。

时间复杂度 $\mathcal{O}(n\log n + 2^{10}\times m)$,具体实现看代码。

```cpp
const int N = 1e6 + 10;
int a[N],n,m,q,ans[N],f[N];
struct node {
    int val,tag;
    int l,r;
}tree[N<<2];

void pushup(int p) {
    tree[p].val = (tree[p << 1].val | tree[p << 1 | 1].val);
}

void pushdown(int p) {
    if (tree[p].tag) {
        tree[p << 1].val = (tree[p].tag);
        tree[p << 1 | 1].val = (tree[p].tag);
        tree[p << 1].tag = tree[p].tag;
        tree[p << 1 | 1].tag = tree[p].tag;
        tree[p].tag = 0;
    }
}

void build(int p,int l,int r) {
    tree[p].l = l,tree[p].r = r;
    if (l == r) {
        tree[p].val = (1 << (a[l]));
        return ;
    }
    int mid = (l + r) >> 1;
    build(p << 1,l,mid);build(p << 1 | 1,mid + 1,r);
    pushup(p);
}

void change(int p,int x,int y,int k) {
    if (tree[p].l >= x && tree[p].r <= y) {
        tree[p].val = (1 << k);
        tree[p].tag = (1 << k);
        return ;
    }
    pushdown(p);
    int mid = (tree[p].l + tree[p].r) >> 1;
    if (x <= mid) change(p << 1,x,y,k);
    if (y > mid) change(p << 1 | 1,x,y,k);
    pushup(p);
}

int query(int p,int x,int y) {
    if (tree[p].l >= x && tree[p].r <= y) {
        return tree[p].val;
    }
    pushdown(p);
    int mid = (tree[p].l + tree[p].r) >> 1,ans = 0;
    if (x <= mid) ans |= query(p <<1,x,y);
    if (y > mid) ans |= query(p << 1 | 1,x,y);
    return ans;
}

struct que {
    int opt,l,r,x;
}Q[N];

vector <int> v;
vector <pii> w[2000];

signed main() {
	read(n,m,q);
    for (int i = 1;i <= n;++i) read(a[i]),v.push_back(a[i]);
    for (int i = 1;i <= q;++i) {
        read(Q[i].opt,Q[i].l,Q[i].r,Q[i].x);
        if (Q[i].opt == 1) v.push_back(Q[i].x);
    }
    sort(v.begin(),v.end());v.erase(unique(v.begin(),v.end()),v.end());
    for (int i = 1;i <= n;++i) a[i] = lower_bound(v.begin(),v.end(),a[i]) - v.begin();
    for (int i = 1;i <= q;++i) if (Q[i].opt == 1) Q[i].x = lower_bound(v.begin(),v.end(),Q[i].x) - v.begin();
    build(1,1,n);
    int cnt= 0;
    for (int i = 1;i <= q;++i) {
        int opt = Q[i].opt,l = Q[i].l,r = Q[i].r,x = Q[i].x;
        if (opt == 1) {
            change(1,l,r,x);
        } else {
            w[query(1,l,r)].push_back(mp(x,++cnt));
        }
    }
    int now = v.size();
    for (int s = 0;s < (1 << now);++s) {
        if (w[s].empty()) continue;
        vector <int> tmp;
        for (int i = 1;i <= now;++i) {
            if (s & (1 << (i - 1))) tmp.push_back(v[i-1]);
        } 
        for (int i = 1;i <= m;++i) f[i] = 0;
        f[0] = 1;
        for (auto x : tmp) {
            for (int i = x;i <= m;++i) {
                f[i] |= f[i-x];
            }
        }
        for (auto p : w[s]){
            if (f[p.first]) ans[p.second] = 1;
            else ans[p.second] = -1;
        }
    }
    for (int i=1;i <= cnt;++i) {
        if (ans[i] == 1) puts("Yes");
        else puts("No");
    }
	return 0;
}
```

# B

我们考虑如果有 $\max\limits_{1 \leq i \leq n} \{L_i\} = \max\limits_{n+1 \leq i \leq n+m} \{L_i\}$，那么矩阵一定存在。这个的证明直接考虑把 $L_{i} = L_{n + j}$ 的位置填最大的值，其他用来填满足其他 $L$ 的值，剩下都填 1 就行了。

那么一个朴素的想法就是钦定一个 $L_{\max}$ 统计答案，答案是显然的。
$$
\sum\limits_{i = 1}^k (i^n - (i-1)^n)\times(i^m - (i - 1)^m)
$$
我们考虑变化一下式子
$$
\begin{aligned}
    &\sum\limits_{i = 1}^k (i^n - (i-1)^n)\times(i^m - (i - 1)^m)\\
    =&\sum\limits_{i = 1}^k i ^{n + m} + (i-1)^{n+m} -i^n(i-1)^m - i^m(i-1)^n
\end{aligned}
$$
这个式子显然是一个 $n+m$ 次多项式。由于我们还求了个和，所以我们只需要把 $n+m+2$ 项给拉格朗日插值一下就好了。
$$
f(x)=\sum\limits_{i=1}^{n+1}y_i\cdot\frac{\prod\limits_{j=1}^{n+1}(x-j)}{(x-i)\cdot(-1)^{n+1-i}\cdot(i-1)!\cdot(n+1-i)!}
$$
时间复杂度 $\mathcal{O}(n\log n )$.

```cpp
const int N = 1e6 + 10;
ll f[N],pre[N],suf[N],fac[N],inv[N];

ll ffpow(ll a,ll b) {
    ll ans = 1;
    for (;b;b >>= 1) {
        if (b & 1) ans = mul(ans,a);
        a = mul(a,a);
    }
    return ans;
}

void solve() {
    ll n,m,k;read(n,m,k);
    ll ans = 0,tot = n + m + 2;
    for (int i = 1;i <= tot;++i) {
        f[i] = add(f[i-1],mul(dec(ffpow(i,n),ffpow(i-1,n)),dec(ffpow(i,m),ffpow(i-1,m))));
    }
    if (k <= tot) ans = f[k];
    else {
        pre[0] = suf[tot + 1] = 1;
        for (int i = 1;i <= tot;++i) pre[i] = mul(pre[i-1],dec(k,i));
        for (int i = tot;i >= 1;--i) suf[i] = mul(suf[i+1],dec(k,i));
        for (int i = 1;i <= tot;++i) {
            ll val = mul(mul(f[i],pre[i-1]),mul(suf[i+1],mul(inv[i-1],inv[tot-i])));
            if ((tot - i) % 2 == 0) ans = add(val,ans);
            else ans = dec(ans,val);
        }
    }
    printf("%lld\n",ans);
}

signed main() {
    fac[0] = inv[0] = inv[1] = fac[1] = 1;
    for (int i = 1;i <= N-2;++i) {
        fac[i] = mul(fac[i-1],i);
    }
    inv[N-2] = ffpow(fac[N-2],mod - 2);
    for (int i = N-2-1;i >= 1;--i) {
        inv[i] = mul(inv[i+1],i+1);
    }
    int T;read(T);
    while (T--) solve();
}
```

# C

## 算法 1

首先我们进行点边转化，我们把每个边的颜色桶覆盖到对应的深度较深的点上，记每个点的颜色桶为 $col_x$。

我们考虑设计一个 DP,令 $f_{x,i}$ 表示对于当次询问，我们钦定点 $x$ 的颜色为 $i(i \in col_x)$，显然我们可以这样转移：
$$
	f_{x,i} = \max\{f_{x-1,j} + [i \not = j]\}
$$
在树上，我们只需要暴力跳找出每个点，把颜色桶单独拉出来做 DP。时间复杂度是 $\mathcal{O}(qnm)$ 的。

## 算法 2

考虑我们可以从两个方面优化这个过程：

1. 颜色数太多是否没有意义？我们是否可以去掉和 $m$ 相关的复杂度？
2. 暴力跳的过程可不可以优化？

要实现第二个方面，首先我们需要对第一个方面进行一些探究。我们对每个点的颜色个数分类考虑：

- $|col_x| > 2$，那么显然我们能找出一个颜色，令 $x$ 和两个相邻的点都是不同颜色的。因此我们对其的讨论就没有意义。
- $|col_x| \leq 2$，这个就是平凡的。

为了方便处理，我们给所有 $|col_x|>2$ 的附上一个新的与其他不同的颜色，将其转化为 $|col_x|=2$ 的情况。

接下来我们考虑原来的 DP 实际上就变成了 $f_{x,0/1}$ 的形式，这个的转移就已经可以通过 $\mathcal{O}(qn)$ 的部分了。考虑继续对其进行优化，我们直接上个倍增，用 $f_{x,k,0/1,0/1}$ 表示点 $x$ 向上跳 $2^k$ 步时，点 $x$ 取第 $0/1$ 个颜色，点 $fa_{x,k}$ 取 $0/1$ 颜色时的最大值。那么这样我们的转移可以通过枚举中间点的颜色将两端拼在一起，也就是这样：
$$
f_{x,k,a,b} = \max\{f_{x,k-1,a,c} + f_{fa_{x,k},k-1,c,b}\}
$$
那么这个东西其实就是 $\mathcal{O}(2^3 q \log n)$ 的了，本题时限较松，可以通过。

```cpp
const int N = 1e6 + 10;
int a[N],f[21][N],dep[N],n,m,q,tot,dp[N][21][2][2],e[N];
vector <int> G[N],col[N];
map <int,int> vis[N],hashc[N];

void dfs1(int x,int fa) {
    f[0][x] = fa;
    dep[x] = dep[fa] + 1;
    for (int i = 1;i <= 20;++i) {
        f[i][x] = f[i-1][f[i-1][x]];
    }
    for (auto y : G[x]) {
        if (y != fa) {
            e[y] = vis[min(x,y)][max(x,y)];
            dfs1(y,x);
        }
    }
}

void dfs2(int x,int fa) {
    //printf("%d %d\n",x,fa);
    if (e[fa]) {
        for (int i = 0;i <= 1;++i) {
            for (int j = 0;j <= 1;++j) {
                if (col[e[x]][i] != col[e[fa]][j]) dp[x][0][i][j] = 1;
            }
        }
    }
    for (int k = 1;k <= 20;++k) {
        for (int i = 0;i <= 1;++i) {
            for (int j = 0;j <= 1;++j) {
                for (int l = 0;l <= 1;++l) {
                    dp[x][k][i][j] = max(dp[x][k][i][j],dp[x][k-1][i][l] + dp[f[k-1][x]][k-1][l][j]);
                }
            }
        }
    }
    for (auto y : G[x]) {
        if (y != fa) {
            dfs2(y,x);
        }
    }
}

inline int LCA(int u, int v) {
	if (u == v) return u;
	if (dep[u] < dep[v]) swap(u, v);
	for (int i = 20;i >= 0;--i) if (dep[u] - (1 << i) >= dep[v]) u = f[i][u];
	if (u == v) return u;
	for (int i = 20;i >= 0;--i) if (f[i][u] ^ f[i][v]) u = f[i][u], v = f[i][v];
	return f[0][u];
}
pair <int,pii > work(int x,int fa) {
    pair <int,pii > ans = mp(x,mp(0,0));
    int now = x;
    for (int i = 20;i >= 0;--i) {
        if (dep[f[i][now]] > dep[fa]) {
            int sf = ans.second.first,ss = ans.second.second;
            ans.second.first = max(sf + dp[now][i][0][0],ss + dp[now][i][1][0]);
            ans.second.second = max(sf + dp[now][i][0][1],ss + dp[now][i][1][1]);
            ans.first = f[i][now];
            now = f[i][now];
        }
    }
    return ans;
}
signed main() {
	read(n,m);
    for (int i = 1;i <= m;++i) {
        int x,y,w;read(x,y,w);
        if (x > y) swap(x,y);
        if (!vis[x][y]) vis[x][y] = ++tot,G[x].push_back(y),G[y].push_back(x);
        if (!hashc[vis[x][y]][w]) hashc[vis[x][y]][w] = 1,col[vis[x][y]].push_back(w);
    }
    for (int i = 1;i <= tot;++i) {
        if (col[i].size() > 2) col[i].clear(),col[i].push_back(m + i),col[i].push_back(m + i);
        else if (col[i].size() == 1) col[i].push_back(col[i][0]);
    }
    dfs1(1,0);dfs2(1,0);
    read(q);
    while (q--) {
        int x,y;read(x,y);
        int lca = LCA(x,y);
        if (x == y) {
            puts("0");
        } else {
            if (lca == x) {
                pair <int,pii > ans = work(y,x);
                printf("%d\n",max(ans.second.first,ans.second.second));
            } else if (lca == y) {
                pair <int,pii > ans = work(x,y);
                printf("%d\n",max(ans.second.first,ans.second.second));
            } else {
                pair <int,pii > ans1 = work(x,lca),ans2 = work(y,lca);
                int ans = max(ans,ans1.second.first + ans2.second.first + (col[e[ans1.first]][0] != col[e[ans2.first]][0]));
                ans = max(ans,ans1.second.first + ans2.second.second + (col[e[ans1.first]][0] != col[e[ans2.first]][1]));
                ans = max(ans,ans1.second.second + ans2.second.first + (col[e[ans1.first]][1] != col[e[ans2.first]][0]));
                ans = max(ans,ans1.second.second + ans2.second.second + (col[e[ans1.first]][1] != col[e[ans2.first]][1]));
                printf("%d\n",ans);
            }
        }
    }
	return 0;
}
```

