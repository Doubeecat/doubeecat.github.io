---
title: CF gym 102798G Caesar Cipher 解题报告
categories: 解题报告
tags: [数据结构,hash]
date: 2021-03-08 13:46:00
---
> [CF gym 102798G](https://codeforces.ml/gym/102798)
> 
> 有一个字符串，每位字符在$[0,65535]$之间。
> 
> 支持两种操作：
> 
> - 将一段区间字符 + 1 后取模 $65536$
> 
> - 询问两段子串是否相同
> 
> $n,Q\leq 500000$

<!--more-->

Sol:

很扎实的数据结构题。

首先你考虑我们平常是怎么求出 hash 值的（某妹子：hash 是啥）

$hash_i = \sum_{j=1}^i hash_{i-1} \times base + a_i$

实际上我们可以把式子转化一下：

$hash_i = \sum_{j=1}^i base^{i-j}$

Code：

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

const int N = 5e5 + 10;
const ll mod = 998244353;
const int MOD = 65536;
const int BASE = 37;

ll pre[N],a[N],n,m;

struct node {
    int l,r;
    ll mmax,hash,sum,tag,base;
}tree[N<<2];

void pushup(int p) {
    tree[p].hash = (tree[p<<1].hash  + tree[p<<1|1].hash) %mod;
    tree[p].mmax = max(tree[p<<1].mmax,tree[p<<1|1].mmax);
}

void build(int p,int l,int r) {
    tree[p].l = l,tree[p].r = r;
    if (l == r) {
        tree[p].mmax = a[l];
        tree[p].hash = (a[l] * pre[l]) % mod;
        tree[p].base = pre[l];
        return ;
    }
    int mid = (l + r) >> 1;
    build(p<<1,l,mid);
    build(p<<1|1,mid + 1,r);
    tree[p].base = (tree[p<<1].base + tree[p<<1|1].base) % mod;
    pushup(p);
}

void pushdown(int p) {
    if (tree[p].tag) {
        tree[p<<1].mmax += tree[p].tag;
        tree[p<<1|1].mmax += tree[p].tag;
        tree[p<<1].tag += tree[p].tag;
        tree[p<<1|1].tag += tree[p].tag;
        tree[p<<1].hash = (tree[p<<1].hash + tree[p].tag * tree[p<<1].base) % mod;
        tree[p<<1|1].hash = (tree[p<<1|1].hash + tree[p].tag * tree[p<<1|1].base) % mod;
        tree[p].tag = 0;
    }
}

void work(int p,int l,int r) {
    if (tree[p].mmax < MOD) return ;
    if (tree[p].l == tree[p].r) {
        if (tree[p].mmax >= MOD) {
            tree[p].mmax -= MOD;
            tree[p].hash = tree[p].sum * pre[l] % mod;
        }
        return ;
    }
    pushdown(p);
    int mid = (tree[p].l + tree[p].r) >> 1;
    if (l <= mid) work(p<<1,l,r);
    if (r > mid) work(p<<1|1,l,r);
    pushup(p);
}

void change(int p,int l,int r) {
    if (tree[p].l >= l && tree[p].r <= r) {
        tree[p].hash = (tree[p].hash + tree[p].base) % mod;
        tree[p].mmax++;
        tree[p].tag++;
        //if (tree[p].mmax >= MOD) work(1,tree[p].l,tree[p].r);
        return ;
    }
    int mid = (tree[p].l + tree[p].r) >> 1;
    pushdown(p);
    if (l <= mid) change(p<<1,l,r);
    if (r > mid) change(p<<1|1,l,r);
    pushup(p);
}

ll query(int p,int l,int r) {
    //printf("%d\n",p);
    if (tree[p].l >= l && tree[p].r <= r) {
        return tree[p].hash;
    }
    pushdown(p);
    int mid = (tree[p].l + tree[p].r) >> 1;
    ll ans = 0;
    if (r > mid) ans = (ans + query(p<<1|1,l,r)) % mod;
    if (l <= mid) ans = (ans + (query(p<<1,l,r)) % mod) % mod;
    return ans;
}

ll debug(int p,int l,int r) {
    //printf("%d\n",p);
    if (tree[p].l >= l && tree[p].r <= r) {
        return tree[p].base;
    }
    pushdown(p);
    int mid = (tree[p].l + tree[p].r) >> 1;
    ll ans = 0;
    if (r > mid) ans = (ans + debug(p<<1|1,l,r)) % mod;
    if (l <= mid) ans = (ans + (debug(p<<1,l,r)) % mod) % mod;
    return ans;
}

signed main() {
    read(n,m);
    pre[0] = 1; 
    for (int i = 1;i <= n;++i) {
        read(a[i]);
        pre[i] = (pre[i-1] * BASE) % mod;
    } 
    build(1,1,n);   
    //for (int i = 1;i <= n;++i) printf("%lld ",debug(1,i,i));
    while (m--) {
        int opt;read(opt);
        if (opt == 1) {
            int l,r;read(l,r);
            change(1,l,r);
            work(1,1,n);
        } else {
            int x1,y1,l;read(x1,y1,l);
            int x2 = x1 + l - 1,y2 = y1 + l - 1;
            ll h1 = (query(1,x1,x2) * pre[y1 - x1]) % mod,h2 = query(1,y1,y2);
            //printf("%lld %lld\n",h1,h2);
            if (h1 == h2) puts("yes");
            else puts("no");
        }/*
        for (int i = 1;i <= n;++i) {
            printf("%lld ",query(1,i,i));
        }
        puts("");*/
    }
	return 0;
}
```