title: CF1638E Colorful Operations 解题报告
categories: 解题报告
tags: [线段树]
date: 2022-02-15 03:59:00
---
> [CF1638E Colorful Operations](https://codeforces.ml/contest/1638/problem/E)
>
> 给定一个长度为 $n$ 的序列，初始时所有元素的值为 $0$ ，颜色为 $1$。你需要实现以下三种操作：
>
> - `Color l r c` ：把 $[l,r]$ 这段的元素颜色改为 $c$
> - `Add c x`：把所有颜色为 $c$ 的元素值都加上 $x$
> - `Query i`：输出元素 $i$ 的值
>
> $n,q \leq 10^6$ 

<!--more-->

## 思路：

这个题有两种思路，一种是树状数组/线段树套珂朵莉树，一种是带 `lazytag` 的线段树，本文在此讲解第二种思路。

首先我们考虑这个把所有颜色为 $c$ 的元素值都加上 $x$ 的操作显然是不好直接处理的，所以我们把这个操作转化为一个全局打标记的操作。具体来说，我们维护一个 $tag_c$ 表示颜色 $c$ 累计加上的值。

接下来考虑单点查询的操作，这个很简单，我们直接线段树上查询元素 $i$ 的值之后加上 $tag_{color_x}$ 即可。

最后考虑区间修改的操作。我们首先考虑 $[l,r]$ 颜色相同的情况，那么只需要把 $[l,r]$ 这段加上原本颜色的 $tag$，消除掉之前的影响，然后再减去新颜色的 $tag$。这里减去新颜色的 $tag$ 是在于我们最后输出时会加上这个 $tag$，并且我们更新的时候实际上也会加上这个 $tag$，所以很巧妙的处理了颜色对于元素的影响。

但是我们发现，如果 $[l,r]$ 颜色不完全相同的时候会不会挂掉？其实是不会的，我们使用颜色段均摊的知识进行证明。

> 对于一个区间颜色连续段，每次线段树上递归修改的复杂度是 $\Theta (\log n)$
>
> 每次区间染色操作，最多增加 $O(1)$ 个区间颜色连续段，所以单次操作均摊复杂度是 $\Theta(\log n)$。

所以我们的代码就能达到  $\Theta(n + q\log n)$ 的复杂度。而对于一个区间连续段是否颜色相同的维护，我们可以直接在 `pushup` 的过程中暴力维护，可以参考以下代码。

## 代码：

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
#define ll long long
#define FO(x) {freopen(#x".in","r",stdin);freopen(#x".out","w",stdout);}
#define pii pair<int,int>
#define mp make_pair
#define int long long

//char buf[1 << 20], *p1, *p2;
//#define getchar() (p1 == p2 && (p2 = (p1 = buf) + fread(buf, 1, 1 << 20, stdin), p1 == p2)?EOF: *p1++)
template <typename T> inline void read(T &t) {
	int v = getchar();T f = 1;t = 0;
	while (!isdigit(v)) {if (v == '-')f = -1;v = getchar();}
	while (isdigit(v)) {t = t * 10 + v - 48;v = getchar();}
	t *= f;
}
template <typename T,typename... Args> inline void read(T &t,Args&... args) {
	read(t);read(args...);
}

const int N = 1e6 + 10;

ll a[N],tag[N],n,q;

struct node {
    int l,r;
    ll tag1,tag2,col,sam;
}tree[N<<2];

void pushup(int p) {
    tree[p].col = tree[p << 1].col;
    tree[p].sam = ((tree[p << 1].sam && tree[p << 1 | 1].sam )&& (tree[p << 1].col == tree[p << 1 | 1].col));
}

void build(int p,int l,int r) {
    tree[p].l = l,tree[p].r = r;
    if (l == r) {
        tree[p].col = 1;
        tree[p].sam = 1;
        return ;
    }
    int mid = (l + r) >> 1;
    build(p << 1,l,mid);
    build(p << 1 | 1,mid + 1,r);
    pushup(p);
}

void pushdown(int p) {
    if (tree[p].tag1) {
        tree[p << 1].tag1 += tree[p].tag1;
        tree[p << 1 | 1].tag1 += tree[p].tag1;
        tree[p].tag1 = 0;
    }
    if (tree[p].tag2) {
        tree[p << 1].col = tree[p << 1 | 1].col = tree[p].tag2;
        tree[p << 1].tag2 = tree[p << 1 | 1].tag2 = tree[p].tag2;
        tree[p << 1].sam = tree[p << 1 | 1].sam = 1;
        tree[p].tag2 = 0;
    }
}

void change(int p,int l,int r,int k) {
    if (tree[p].l >= l && tree[p].r <= r && tree[p].sam) {
        tree[p].tag1 += tag[tree[p].col];
        tree[p].col = k;
        tree[p].tag2 = k;
        tree[p].sam = 1;
        tree[p].tag1 -= tag[k];
        return ;
    }
    int mid = (tree[p].l + tree[p].r) >> 1;
    pushdown(p);
    if (l <= mid) change(p << 1,l,r,k);
    if (r > mid) change(p << 1 | 1,l,r,k);
    pushup(p);
}

ll query(int p,int x) {
    //printf("%lld %lld %lld %lld %lld\n",p,tree[p].l,tree[p].r,tree[p].tag2,tree[p].col);
    if (tree[p].l == tree[p].r) {
        return tree[p].tag1 + tag[tree[p].col];
    }
    int mid = (tree[p].l + tree[p].r) >> 1;
    pushdown(p);
    if (x <= mid) return query(p << 1,x);
    else return query(p << 1 | 1,x);
}

signed main() {
    read(n,q);
    build(1,1,n);
    for (int i = 1;i <= q;++i) {
        char opt[10];scanf("%s ",opt);
        if (opt[0] == 'C') {
            int x,y,w;read(x,y,w);
            change(1,x,y,w);
        }
        if (opt[0] == 'A') {
            int x,k;read(x,k);
            tag[x] += k;
        }
        if (opt[0] == 'Q') {
            int x;read(x);
            printf("%lld\n",query(1,x));
        }
    }
	return 0;
}
```

