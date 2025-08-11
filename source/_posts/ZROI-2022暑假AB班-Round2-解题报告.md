---
title: ZROI 2022暑假AB班 Round2 解题报告
categories: 解题报告
tags: [线段树,ETT,博弈论,nim游戏,线性基]
date: 2022-08-09 14:40:00
---
100 + 10 + 0，rk 21

<!--more-->

# A

说点闲话：

![image-20220809153844683](https://s2.loli.net/2022/08/09/OkqZrENXMUyLpiP.png) <img src="https://s2.loli.net/2022/08/09/rQJCtfBn63GbcZh.png" alt="image-20220809153855336" style="zoom:80%;" />



了转反。

原题：[gym103069G](https://codeforces.ml/gym/103069/problem/G)

借此机会概括一类区间子区间问题的思考方法。

对于这类区间的题，我们首先考虑扫描线，把整个二维区间当成一个二维平面上的点丢到平面直角坐标系上，每个点 $(x,y)$ 代表的是区间 $[x,y]$ ，注意我们这边有一个 $x \leq y$ 的性质，所以这个平面存在的点实际上构成了一个上三角。

那么对于一个 $[l,r]$ 的查询，就是查询一个上三角里满足条件的点的个数。利用扫描线思想，我们转化为对于 $y$ 轴上一个区间从 $x = 1$ 到当前有多少个位置满足条件，差分一下就可以了。

![image-20220809223337900](https://s2.loli.net/2022/08/09/7FXAkGKTBJIQalN.png)

用一个数据结构维护历史信息就可。我们对每个位置 $i$ 维护一个是否合法的值（这题中就是合法后缀数量和不合法后缀数量） $a_i$，当扫描线扫到 $r$ 的时候，找到最大的 $i$ 满足 $a_r =a_i,i < r$，则说明当前右端点下左区间在端点 $[i+1,r]$ 中，额外出现了颜色 $col_r$.

注意到求奇数个颜色的可以转化为区间色数 $\bmod 2$

线段树维护每个左端点对应区间的异或和，区间多一个颜色那就是区间 $\text{xor } 1$ 。

查询等价于对于右端点 $r^{\prime} \in [l,r]$,查询一个左端点 $l^{\prime} \in [l,r]$ 的区间和答案，这个就是我们上边说的差分后转化为查询 $[l,r]$ 在 $r$ 时刻的历史和，这个也简单，在更新的时候维护即可。

时间复杂度 $\mathcal{O}((n +m)\log n)$

```cpp
//Every night that summer just to seal my fate
//And I scream, "For whatever it's worth
//I love you, ain't that the worst thing you ever heard?"
#include <algorithm>
#include <iostream>
#include <cstring>
#include <cstdio>
#include <cctype>
#include <vector>
#include <cmath>
#include <queue>
using namespace std;
#define ll long long
#define FO(x) {freopen(#x".in","r",stdin);freopen(#x".out","w",stdout);}
#define pii pair<int,int>
#define mp make_pair

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

const ll mod = 998244353;
const double eps = 1e-10;
const int N = 5e5 + 10;
int n,a[N],pre[N],m;

struct node {
    int l,r;
    ll reve,tag0,tag1;
    ll cnt0,cnt1,sum;
}tree[N<<2];
//reve reverse tag0 pushdown to 0 tag1 pushdown to 1
//tot ->cnt of 1 cntof0->len-tot
void pushup(int p) {
    tree[p].sum = tree[p << 1].sum + tree[p << 1 | 1].sum;
    tree[p].cnt1 = tree[p << 1].cnt1 + tree[p << 1 | 1].cnt1;
    tree[p].cnt0 = tree[p << 1].cnt0 + tree[p << 1 | 1].cnt0;
}

void pushdown(int p) {
    if (tree[p].reve) {
        swap(tree[p << 1].cnt0,tree[p << 1].cnt1);
        swap(tree[p << 1 | 1].cnt0,tree[p << 1 | 1].cnt1);

        swap(tree[p << 1].tag0,tree[p << 1].tag1);
        swap(tree[p << 1 | 1].tag0,tree[p << 1 | 1].tag1);
        tree[p << 1].reve ^= 1;tree[p << 1 | 1].reve ^= 1;
        tree[p].reve = 0;
    }
    if (tree[p].tag0) {
        tree[p << 1].sum += tree[p].tag0 * tree[p << 1].cnt0;
        tree[p << 1 | 1].sum += tree[p].tag0 * tree[p << 1 | 1].cnt0;
        tree[p << 1].tag0 += tree[p].tag0;
        tree[p << 1 | 1].tag0 += tree[p].tag0;
        tree[p].tag0 = 0;
    }
    if (tree[p].tag1) {
        tree[p << 1].sum += tree[p].tag1 * (tree[p << 1].cnt1);
        tree[p << 1 | 1].sum += tree[p].tag1 * (tree[p << 1 | 1].cnt1);
        tree[p << 1].tag1 += tree[p].tag1;
        tree[p << 1 | 1].tag1 += tree[p].tag1;
        tree[p].tag1 = 0;
    }
}

void build(int p,int l,int r) {
    tree[p].l = l,tree[p].r = r;
    if (l == r) {
        tree[p].cnt0 = 1;
        return ;
    }
    int mid = (l + r) >> 1;
    build(p << 1,l,mid);
    build(p << 1 | 1,mid + 1,r);
    pushup(p);
}

void change(int p,int x,int y) {
    if (tree[p].l >= x && tree[p].r <= y) {
        swap(tree[p].cnt0,tree[p].cnt1);
        swap(tree[p].tag0,tree[p].tag1);
        tree[p].reve ^= 1;
        return ;
    }
    pushdown(p);
    int mid = (tree[p].l + tree[p].r) >> 1;
    if (x <= mid) change(p << 1,x,y);
    if (y > mid) change(p << 1 | 1,x,y);
    pushup(p);
} 

ll query(int p,int x,int y) {
    if (tree[p].l >= x && tree[p].r <= y) {
        return tree[p].sum;
    }
    pushdown(p);
    int mid = (tree[p].l + tree[p].r) >> 1;
    ll ans = 0;
    if (x <= mid) ans += query(p << 1,x,y);
    if (y > mid) ans += query(p << 1 | 1,x,y);
    return ans;
}

struct que {
    int l,r,num;
    friend inline bool operator < (const que &a,const que &b) {
        return a.r == b.r ? a.l < b.l : a.r < b.r;
    }
}q[N];

int lst[N];
ll ans[N];

signed main() {
    read(n);
    for (int i = 1;i <= n;++i) read(a[i]);
    build(1,1,n);
    read(m);
    for (int i = 1;i <= m;++i) {
        read(q[i].l,q[i].r);q[i].num = i;
    }
    sort(q + 1,q + 1 + m);
    int now = 1;
    for (int i = 1;i <= m;++i) {
        while (now <= q[i].r) {
            change(1,lst[a[now]] + 1,now);
            tree[1].sum += (tree[1].cnt1);
            tree[1].tag1 ++;
            lst[a[now]] = now;
            ++now;
        }
        ans[q[i].num] = query(1,q[i].l,q[i].r);
    }
    for (int i = 1;i <= m;++i) printf("%lld\n",ans[i]);
	return 0;
}
```



# B

前置结论：k-Nim 游戏，线性基。

我们考虑这个游戏本质上是一个 k-Nim 游戏 $k = 2$ 的情况，那么有结论：

> 先手必败，当且仅当将每个 $a_i$ 写成二进制，对于每一个二进制位，这一位为 1 的 $i$ 的个数为 $s$，$s\bmod (k+1) = 0$.
>
> 证明类似普通的 nim 游戏，加起来为 0 的下一 步一定不为 0，不为 0 的一定存在一种选法使得能变成 0。

在 $b_i = 1$ 的条件下，我们的原问题就变成了：求一个最大子集 $A$，且 $A$ 在 $\bmod 3$ 意义下线性无关。那么这个就可以用一个三进制线性基来维护。

对于三进制线性基来说，我们要维护的就是 $\bmod3$ 意义下的异或来代替原来的线性基。我们可以这么实现：

```cpp
struct sint {
    int a[M],k = 3;

    sint(int x = 0) {
        memset(a,0,sizeof a);
        for (int i = 0;i < M;++i) {
            a[i] = (x >> i &1);
        }
    }

    friend inline sint operator ^ (const sint &A,const sint &B) {
        sint C;
        for (int i = 0;i < M;++i) C.a[i] = (A.a[i] + B.a[i]) % 3;
        return C;
    }

    friend inline sint operator - (const sint &A,const sint &B) {
        sint C;
        for (int i = 0;i < M;++i) C.a[i] = (A.a[i] - B.a[i] + 3) % 3;
        return C;
    }
} s[M];

ll n,c[N];

void insert(int x,ll val) {
    sint now(x);
    for (int i = M-1;i >= 0;--i) {
        if (now.a[i]) {
            if (!s[i].a[i]) {
                s[i] = now;
                c[i] = val;
                return ;
            } else {
                if (val > c[i]) {swap(c[i],val),swap(s[i],now);}
                if (now.a[i] == s[i].a[i]) now = now - s[i];
                else now = now - s[i] - s[i];
            }
        }
    }
}

int query() {
    int ans = 0;
    for (int i = M-1;i >= 0;--i) ans += c[i];
    return ans;
}

signed main() {
    read(n);
    int lst = 0;
    for (int i = 1;i <= n;++i) {
        int a,b;read(a,b);a ^= lst;
        insert(a,b);
        printf("%d\n",lst = query());
    }
	return 0;
}

```

# C

首先考虑 $p_i = 0$，一个灯对应两个开关，那么显然是在这两个开关连边，而每个 $i$ 往更小的 $i$ 连边，显然构成的是一个森林！

最后的工作就变成了给点染色，那么我们从根考虑，只要让点的颜色都和根相同就可以了。一共两种方案讨论一下就行。

我们考虑如何维护修改，实际上就是把一个子树接到了另一颗树上，这个 LCT很难实现，所以我们引入一种数据结构 ETT（[Euler Tour Tree - OI Wiki (oi-wiki.org)](https://oi-wiki.org/ds/ett/)）。

ETT 核心是用括号序列刻画一个树，上述接树的操作我们可以用一颗 FHQ-Treap 来做，接下来考虑需要维护的信息：对于每个节点维护两个值：括号异或和，前缀异或和为 0 的数量。这两个是简单合并的，发现一个节点被合并两次，那么和根节点颜色一样的数就是前缀为 $0$ 的数量的一半。