---
title: CF gym 102920 I Stock Analysis 解题报告
categories: 解题报告
tags: [数据结构,树状数组]
date: 2021-03-05 15:24:31
---

> CF gym 102920 I
> 
> 给出一个序列和若干询问，每次求 $[l,r]$ 的子区间中不超过 $w$ 的最大区间和。
> 
> $n\leq 2000,Q\leq 200000$

<!--more-->

Sol:

首先这个范围就不是很数据结构，这个询问反倒很数据结构。

我们可以发现，其实一共最多只有 $\frac{n\times (n-1)}{2}$ 个子区间，我们大可以先把这些子区间取出来，对于大小排序，然后对于每个询问依次考虑。

我们考虑对于每个位置 $x$ 维护一个树状数组，这里维护的是以 $x$ 为左端点的子区间的最大值。

然后我们发现实际上是没有 $y < x$ 的情况的，所以我们可以直接多开一维，然后按照普通维护树状数组的方法维护一个前缀最大值即可。

```cpp
struct BIT{
    ll p[N][N];
 
    void init() {
        for (int i = 1;i <= n;++i) {
            for (int j = 0;j <= n;++j) {
                p[i][j] = -INF;
            }
        }
    }
 
    int lowbit(int x) {return x & (-x);}
 
    void modify(int x,int y,ll w) {
        for (;y <= n;y += lowbit(y)) {
            p[x][y] = max(p[x][y],w);
        }
    }
 
    ll query(int x,int y) {
        ll ans = -INF;
        for (;y;y -= lowbit(y)) {
            ans = max(ans,p[x][y]);
        }
        return ans;
    }
};
```

然后这样其实就可以通过了。每次的时间复杂度是 $O(\log n)$。

但是我们怎么能止步于此呢！！！！（其实是算复杂度的时候人算傻了）

我们考虑！对序列进行分块，然后再额外开一个树状数组维护每一块内以其中某个点为左端点的子区间的最大值。

每次询问的时候，$l,r$ 同一块的话就直接块内暴力查询，否则的话先暴力处理左右两块的答案，中间直接跳即可。

时间复杂度 $O(\sqrt{n}\log n)$ 吧，看着好像非常nb，实际上也就比上一个方法快了100多ms，血亏

Code:

直接树状数组啥事没有

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


const int N = 4010;
const int M = 4e6 + 10;
const ll INF = 1e17;

int n,m;

ll p[N][N];

int lowbit(int x) {return x & (-x);}

void modify(int x, int y, long long v){
    for (int i = x; i < N; i += lowbit(i)){
        for (int j = y; j < N; j += lowbit(j)){
            p[i][j] = max(p[i][j], v);
        }
    }
}

long long query(int x, int y){
    long long v = -1e18;
    for (int i = x; i > 0; i -= lowbit(i)){
        for (int j = y; j > 0; j -= lowbit(j)){
            v = max(v, p[i][j]);
        }
    }
    return v;
}


ll ans[M],st[N],prem;

struct memb {
    int l,r,pos;
    ll w;
    friend inline bool operator <(const memb &a,const memb &b) {
        return a.w == b.w ? a.pos < b.pos : a.w < b.w;
    }
}q[M];

std::vector<pair<pair<ll,int>,pair<int,int> > > v;

ll a[N];

signed main() {
    ios::sync_with_stdio(false);
    for (int i = 0;i < N;++i) {
        for (int j = 0;j < N;++j) {
            p[i][j] = -INF;
        }
    }
    cin >> n >> m;
    for (int i = 1;i <= n;++i) cin >> a[i];
    for (int i = 1;i <= m;++i) {
        int x,y;ll w;
        cin >> x >> y >> w;
        v.push_back(make_pair(make_pair(w,i),make_pair(x,y)));
    }
    for (int i = 1;i <= n;++i) {
        ll tmp = 0;
        for (int j = i;j <= n;++j) {
            tmp += a[j];
            v.push_back(make_pair(make_pair(tmp,-1),make_pair(i,j)));
        }
    }
    sort(v.begin(),v.end());
    for (auto p:v) {
        //cout << q[i].l<< " " << q[i].r<< " "<< q[i].pos << " " << q[i].w << endl;
        if (p.first.second != -1) ans[p.first.second] = query(n - p.second.first + 1,p.second.second);
        else modify(n - p.second.first + 1,p.second.second,p.first.first);
    }   
    for (int i = 1;i <= m;++i) {
        if (ans[i] != -INF) cout << ans[i] << endl;
        else cout << "NONE" << endl;
    }
	return 0;
}
```

啥比才写分块

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

const int N = 4010;
const int M = 4e6 + 10;
const ll INF = 1e17;

int n,m;

ll p[N][N];

void init() {
    for (int i = 0;i < N;++i) {
        for (int j = 0;j < N;++j) {
            p[i][j] = -INF;
        }
    }
}

int lowbit(int x) {return x & (-x);}

void modify(int x, int y, long long v){
    for (int i = x; i < N; i += lowbit(i)){
        for (int j = y; j < N; j += lowbit(j)){
            p[i][j] = max(p[i][j], v);
        }
    }
}

long long query(int x, int y){
    long long v = -1e18;
    for (int i = x; i > 0; i -= lowbit(i)){
        for (int j = y; j > 0; j -= lowbit(j)){
            v = max(v, p[i][j]);
        }
    }
    return v;
}


ll ans[M],st[N],prem;

struct memb {
    int l,r,pos;
    ll w;
    friend inline bool operator <(const memb &a,const memb &b) {
        return a.w == b.w ? a.pos < b.pos : a.w < b.w;
    }
}q[M];


ll a[N];

signed main() {
    read(n,m);
    for (int i = 1;i <= n;++i) read(a[i]);
    for (int i = 1;i <= m;++i) {
        read(q[i].l,q[i].r);read(q[i].w);q[i].pos = i;
    }
    prem = m;
    init();
    for (int i = 1;i <= n;++i) {
        ll tmp = 0;
        for (int j = i;j <= n;++j) {
            tmp += a[j];
            q[++m].l = i;q[m].r = j;q[m].w = tmp;q[m].pos = -1;
        }
    }
    sort(q+1,q+1+m);
    for (int i = 1;i <= m;++i) {
        //cout << q[i].l<< " " << q[i].r<< " "<< q[i].pos << " " << q[i].w << endl;
        if (q[i].pos != -1) ans[q[i].pos] = query(n - q[i].l + 1,q[i].r);
        else modify(n - q[i].l + 1,q[i].r,q[i].w);
    }   
    for (int i = 1;i <= prem;++i) {
        if (ans[i] != -INF) cout << ans[i] << endl;
        else puts("NONE");
    }
	return 0;
}
```