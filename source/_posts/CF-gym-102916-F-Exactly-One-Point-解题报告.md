---
title: CF gym 102916 F Exactly One Point 解题报告
categories: 解题报告
tags: [DP,数据结构,差分约束]
date: 2021-03-04 13:44:50
---


> [CF gym 102916 F](https://codeforces.com/gym/102916/problem/F)
> 
> 数轴上有若干线段，请在数轴上放置若干点，满足：
> 
> 每个线段恰好包含一个点
> 
> 每个点至少被一个线段所包含
> 
> $n\leq 200000$


<!--more-->

Sol:

考虑差分约束，差分约束不会的可以先看[这个](l,r+1,1),add(r+1,l,-1)我们记 $num_i$ 为第 $i$ 个位置放了几个点，那么显然，我们可以得出以下约束条件：

- 每个线段恰好包含一个点，也就是 $num_{r-1} - num_l = 1$，也就是建两条边：$l \rightarrow r+1(w = 1)$,$r+1 \rightarrow l (w = -1)$

- 每个点只能被选一次，那么我们需要人为对点进行限制，也就是 $num_i - num_{i-1} \leq 1,num_{i-1} - num_i \leq 0 $
	也就是建边 $i-1 \rightarrow i(w=0),i \rightarrow i-1(w = 1)$

但是这个题 SPFA 是过不去的，所以你要把 SPFA 换成 Dijkstra。并且为了解决负环的问题你需要**小小**魔改一下：

```cpp
void Dijkstra() {
    memset(dis,-1,sizeof dis);
    dis[0] = 0;q.push({0,0});
    while (!q.empty()) {
        pii p = q.top();q.pop();
        int x = p.second;
        if (p.first != dis[x]) continue;//魔改地方
        for (int i = hd[x];i;i = nxt[i]) {
            int y = to[i],w = edg[i];
            if (dis[x] + w > dis[y]) {
                dis[y] = dis[x] + w;
                q.push({dis[y],y});
                if (++cnt >= (ll)1e7) {//魔改地方
                    puts("-1");exit(0);
                }
            }
        }
    }
}
```

可以看到，这样魔改完我们每个点不止经过一次，但是我们可以人为判断一下我们是否做了很多次松弛操作（有负环不成立），直接排除掉即可，可以通过本题。

下面介绍一下本题的正经做法（但是我觉得比差分约束难想&写很多）:

首先我们考虑 DP，设 $f_i$ 为在 $i$ 放点的话它是由 $f_i$ 转移来的，然后如果没有前置点则 $f_i = -1$。然后我们这个 DP 是从左到右顺推过去的，所以我们在处理当前这个点的时候必须考虑下一个点放在哪里。

为了做到这点，我们先预处理出两个量：

- $maxR_x$，表示对于所有 $l\leq x$ 的线段的最大右端点，用 set 可以随手维护。

- $Next_x$ 表示对于所有 $l > x$ 的线段中有最小右端点的线段，这个和上面一样用 set 维护即可。

接下来我们来考虑一下转移的范围，它和这两个量息息相关。

首先我们肯定不能放在小于等于 $maxR_x$ 的地方，因为我们已经放了一个点在 $x$ 了，然后最长的包含 $x$ 的线段的右端点是在 $maxR_x$。

其次，我们肯定不能放在大于 $next_x$ 的右端点的地方，因为这样这个线段就会空出来。

所以我们就得出了我们的转移范围 $(maxR_x,next_x.r]$。

对于这个转移，我们实际上可以用一颗线段树维护出这个区间内为 0 的点，然后我们将这个为 0 的点所在的线段进行一个区间修改。

每个线段只会被操作到一次，所以时间复杂度是严格的 $O(n\log n)$

Code:

乱搞：

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
#define ri register int

char buf[1 << 20], *p1, *p2;
//#define getchar() (p1 == p2 && (p2 = (p1 = buf) + fread(buf, 1, 1 << 20, stdin), p1 == p2)?EOF: *p1++)
template <typename T> inline void read(T &t) {
	ri v = getchar();T f = 1;t = 0;
	while (!isdigit(v)) {if (v == '-')f = -1;v = getchar();}
	while (isdigit(v)) {t = t * 10 + v - 48;v = getchar();}
	t *= f;
}
template <typename T,typename... Args> inline void read(T &t,Args&... args) {
	read(t);read(args...);
}

const int N = 2e6 + 10;
#define pii pair<int,int>

priority_queue <pii> q;

int hd[N],nxt[N],to[N],edg[N],tot,n,num[N],dis[N],cnt;
bool vis[N],flag;

inline void add(int u,int v,int w) {to[++tot] = v;nxt[tot] = hd[u];hd[u] = tot;edg[tot] = w;}

void Dijkstra() {
    memset(dis,-1,sizeof dis);
    dis[0] = 0;q.push({0,0});
    while (!q.empty()) {
        pii p = q.top();q.pop();
        int x = p.second;
        if (p.first != dis[x]) continue;
        for (int i = hd[x];i;i = nxt[i]) {
            int y = to[i],w = edg[i];
            if (dis[x] + w > dis[y]) {
                dis[y] = dis[x] + w;
                q.push({dis[y],y});
                if (++cnt >= (ll)1e7) {
                    puts("-1");exit(0);
                }
            }
        }
    }
}

signed main() {
    read(n);
    for (int i = 1;i <= 4*n-1;++i) {
        add(i-1,i,0);
        add(i,i-1,-1);
    }
    for (int i = 1;i <= n;++i) {
        int l,r;read(l,r);
        add(l,r+1,1),add(r+1,l,-1);
    }
    Dijkstra();
    int ans = 0;
    for (int i = 1;i <= 4*n-1;++i) {
        if (dis[i] > dis[i-1]) num[++ans] = i-1;
    }
    printf("%d\n",ans);
    for (int i = 1;i <= ans;++i) printf("%d ",num[i]);
	return 0;
}
```