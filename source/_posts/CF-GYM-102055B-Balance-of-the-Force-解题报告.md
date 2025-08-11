---
title: CF GYM 102055B Balance of the Force 解题报告
categories: 解题报告
tags: [二分图]
date: 2021-06-28 02:18:00
---
>有 $n$ 个骑士，每个骑士可以加入光阵营或暗阵营，并且获得一个力量，分别为 $L$ 和 $D$
>
>现在有 $m$ 对骑士不能在同一个阵营，问力量差值（最大值和最小值的差值）最小是多少？如果不存在阵营满足仇恨骑士不在同一个阵营内，则输出IMPOSSIBLE。
>
>$n,m \leq 10^5$

<!--more-->

首先判断一下这张图是不是二分图，如果不是显然无解。

接下来我们考虑同一个联通块内答案显然只有两种，分别弄出来。

接下来的思路比较重要，到了这一步已经划归为一个类似偏序的问题，那我们显然先排序一下最大值，考虑最小值怎么搞

实际上变成了在一个范围内查询最小值，然后这个问题可以用线段树解决。

但是实际上这个题并不好写，又臭又长，写了三个小时左右

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
#define ri int

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

vector <int> G[N];
int n,m,l[N],d[N],col[N],qwq,cnt,tot,pre[N],pos[N];
bool vis[N];

struct memb {
    int minn,maxx,bel;
    friend inline bool operator < (const memb &a1,const memb &a2) {
        return a1.maxx < a2.maxx;
    }
}a[N];

bool dfs1(int x,int c) {
    col[x] = c;
    for (auto y : G[x]) {
        if (col[y] == c) return false;
        if (!col[y] && !dfs1(y,3-c)) return false;
    }
    return 1;
}

bool check() {
    for (int i = 1;i <= n;++i) {
        if (!col[i]) {
            if (!dfs1(i,1)) return false;
        }
    }
    return 1;
} 

void color(int x,int c,int las) {
    //printf("%d %d %d\n",x,c,las);
    col[x] = c;
    //a[i] darkside/lightside a[i+1] lightside/darkside
    if (las == 0) {
        a[tot].minn = min(d[x],a[tot].minn);
        a[tot].maxx = max(d[x],a[tot].maxx);
        a[tot-1].minn = min(l[x],a[tot-1].minn);
        a[tot-1].maxx = max(l[x],a[tot-1].maxx);
    } else {
        a[tot].minn = min(l[x],a[tot].minn);
        a[tot].maxx = max(l[x],a[tot].maxx);
        a[tot-1].minn = min(d[x],a[tot-1].minn);
        a[tot-1].maxx = max(d[x],a[tot-1].maxx);
    }
    for (auto y : G[x]) {
        if (!col[y]) color(y,c,las^1);
    }
}

struct node {
    int l,r,minn;
}tree[N<<2];

void build(int p,int l,int r) {
    tree[p].l = l;tree[p].r = r;
    if (l == r) {
        tree[p].minn = 0x3f3f3f3f;
        return ;
    }
    int mid = (l + r) >> 1;
    build(p << 1,l,mid);
    build(p << 1 | 1,mid + 1,r);
    tree[p].minn = min(tree[p << 1].minn,tree[p<<1|1].minn);
    return ;
}

void modify(int p,int x,int v) {
    if (tree[p].l == tree[p].r ) {
        tree[p].minn = v;
        return ;
    }
    int mid = (tree[p].l + tree[p].r) >> 1;
    if (x <= mid) modify(p<<1,x,v);
    else modify(p<<1|1,x,v);
    tree[p].minn = min(tree[p << 1].minn,tree[p<<1|1].minn);
    return ;
}

void solve() {
    printf("Case %d: ",qwq);
    tot = cnt = 0;
    read(n,m);
    for (int i = 1;i <= n;++i) G[i].clear(),vis[i] = 0,pos[i] = 0;
    for (int i = 1; i <= m;++i) {
        int x,y;read(x,y);
        G[x].push_back(y);G[y].push_back(x);
    }
    for (int i = 1;i <= n;++i) {
        read(l[i],d[i]);
        col[i] = 0;
    }
    if (!check()) {
        puts("IMPOSSIBLE");
        return ;
    }
    for (int i = 1;i <= n;++i) col[i] = 0;
    for (int i = 1;i <= n;++i) {
        if (!col[i]) {
            tot += 2;
            a[tot].minn = 0x3f3f3f3f,a[tot].maxx = -0x3f3f3f3f;
            a[tot-1].minn = 0x3f3f3f3f,a[tot-1].maxx = -0x3f3f3f3f;
            a[tot].bel = a[tot-1].bel = ++cnt;
            color(i,cnt,0);
        }
        //printf("%d:%d\n",i,col[i]);
    }//calc every CC
    //puts("part1");
    sort(a+1,a+tot+1);
    /*for (int i = 1;i <= tot;++i) {
        printf("%d %d %d\n",a[i].minn,a[i].maxx,a[i].bel);
    }*/
    pre[0] = 0;
    for (int i = 1;i <= tot;++i) {
        pre[i] = pre[i-1];
        if (!vis[a[i].bel]) {
            pre[i]++;vis[a[i].bel] = 1;
        }
    }//sort and calc every CC
    //puts("part2");
    build(1,1,tot);
    //puts("part3");
    int curmax,curmin = 0x3f3f3f3f;
    int ans = 0x3f3f3f3f;
    for (int i = 1;i <= tot;++i) {
        //printf("%d ",pre[i]);
        curmax = a[i].maxx,curmin = a[i].minn;
        if (pre[i] == cnt) {
            if (pos[a[i].bel]) {
                modify(1,pos[a[i].bel],0x3f3f3f3f);
            }
            curmin = min(curmin,tree[1].minn);
            //printf("%d %d\n",curmin,i);
            ans = min(ans,curmax - curmin);
        }
        if (pos[a[i].bel]) {
            int tmp = max(a[i].minn,a[pos[a[i].bel]].minn);
            modify(1,pos[a[i].bel],tmp);
            modify(1,i,tmp);
        } else {
            pos[a[i].bel] = i;
            modify(1,i,a[i].minn);
        }
    //    puts("part4");
    }//for every max num,find a closest num to fix it
    printf("%d\n",ans);
}

signed main() {
	int T;read(T);
    for (qwq = 1;qwq <= T;++qwq) solve();
    return 0;
}
```