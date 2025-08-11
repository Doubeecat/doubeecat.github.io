---\r\title: Luogu P1772 [ZJOI2006]物流运输 解题报告
categories: 解题报告
tags: [图论,最短路,DP,区间DP]
date: 2020-09-05 03:39:00
---
> [P1772 [ZJOI2006]物流运输](https://www.luogu.com.cn/problem/P1772)
> 
> 物流公司要把一批货物从码头 1 运到码头 $m$。由于货物量比较大，需要 $n$ 天才能运完，共有 $m$ 个码头。
> 
> 物流公司会设计一条固定的运输路线，有的时候某个码头会无法装卸货物。这时候就必须修改运输路线，让货物能够按时到达目的地。
> 
> 一次修改路线会带来 $k$ 的成本。因此物流公司希望能够订一个 $n$ 天的运输计划，使得总成本尽可能地小。
> 
> $n \leq 100,m \leq 20$
<!--more-->

## 解题思路：

首先观察下这个题的样例解释，可以发现一个非常重要的思路：

每一个时间段中，**不调整路线**的最短路显然是固定的。

所以我们可以先考虑处理每个时间区间的**不调整路线**的最短路总长度 $f_{l,r}$。

发现全部都是正权边，我们可以使用 Dijkstra 来进行这一过程，只需要在每次松弛时将该区间里被封锁的点去掉即可。

注意，因为我们是对整个时间区间进行最短路处理，所以最后的长度要乘上 $r - l + 1$。

代码实现：
```cpp

read(z);
for (int i = 1;i <= z;++i) {
    int p,a,b;read(p,a,b);
    for (int j = a;j <= b;++j) {
        rea[j][p] = 1;
    }//rea[i][j]表示第i天点j是否被封锁
}

void dijkstra(int l,int r) {
    memset(dis,0x3f,sizeof dis);
    memset(vis,0,sizeof vis);
    memset(now,0,sizeof now);
    priority_queue <node> q;
    for (int i = l;i <= r;++i) {
        for (int j = 1;j <= m;++j) {
            if (rea[i][j]) now[j] = 1;
        }//如果在l - r 里这个点被封锁过，那么我们就不考虑这个点
    }
    dis[1] = 0;
    q.push((node){0,1});
    while (!q.empty()) {
        node p = q.top();q.pop();
        int x = p.num;
        vis[x] = 1;
        for (int i = hd[x];i;i = nxt[i]) {
            int y = to[i];
            if (!vis[y] && !now[y]) {//松弛的时候多加一步判断即可
                if (dis[y] > dis[x] + edg[i]) {
                    dis[y] = dis[x] + edg[i];
                    q.push((node){dis[y],y});
                }
            }
        }
    }
    if (dis[m] != INF) f[l][r] = dis[m] * (r - l + 1);
    //第 l - r 天的 最短路总长度
    else f[l][r] = INF;
}
```

接下来处理一个问题，我们要求的显然是 $f_{1,n}$，但是就如样例那样不连通怎么办？那就需要更改路径了。

注意到，第 $l\dots r$ 天的路径实际上可以先走 $l\dots p$ 的路径，再花费 $k$ 的代价更换到 $p+1 \dots r$ 的路径得到。

换句话说，这个代价事实上满足大段最优解是小段最优解合并来的性质，而这正是 **区间 DP** 所要求的。

并且这里 $k$ 不变，所以直接利用区间 DP 计算即可。

总时间复杂度 $O(n^2 m \log m + n ^ 3)$

## 代码：

```cpp
const int INF = 0x3f3f3f3f;
const int N = 200;
const int M = 400;
int dis[N],f[N][N],d[N][N],n,m,rr,e,z;

int hd[N],nxt[M],edg[M],to[M],tot;
bool vis[N],rea[N][N],now[N];

void add(int u,int v,int w) {to[++tot] = v,edg[tot] = w,nxt[tot] = hd[u],hd[u] = tot;}
void addedge(int u,int v,int w) {add(u,v,w),add(v,u,w);}

struct node {
    int dis,num;
    friend inline bool operator < (const node & a,const node &b) {
        return b.dis < a.dis;
    }
};

void dijkstra(int l,int r) {
    memset(dis,0x3f,sizeof dis);
    memset(vis,0,sizeof vis);
    memset(now,0,sizeof now);
    priority_queue <node> q;
    for (int i = l;i <= r;++i) {
        for (int j = 1;j <= m;++j) {
            if (rea[i][j]) now[j] = 1;
        }
    }
    dis[1] = 0;
    q.push((node){0,1});
    while (!q.empty()) {
        node p = q.top();q.pop();
        int x = p.num;
        vis[x] = 1;
        for (int i = hd[x];i;i = nxt[i]) {
            int y = to[i];
            if (!vis[y] && !now[y]) {
                if (dis[y] > dis[x] + edg[i]) {
                    dis[y] = dis[x] + edg[i];
                    q.push((node){dis[y],y});
                }
            }
        }
    }
    if (dis[m] != INF) f[l][r] = dis[m] * (r - l + 1);
    else f[l][r] = INF;
}

void input() {
    read(n,m,rr,e);
    for (int i = 1;i <= e;++i) {
        int u,v,w;read(u,v,w);
        addedge(u,v,w);
    }
    read(z);
    for (int i = 1;i <= z;++i) {
        int p,a,b;read(p,a,b);
        for (int j = a;j <= b;++j) {
            rea[j][p] = 1;
        }
    }
}

void DP() {
    for (int len = 2;len <= n;++len) {
        for (int i = 1;i + len - 1 <= n;++i) {
            int j = i + len - 1;
            for (int k = i;k <= j;++k) {
                f[i][j] = min(f[i][j],f[i][k] + f[k+1][j] + rr);
            }
        }
    }
}

signed main() {
    input();
    for (int i = 1;i <= n;++i) {
        for (int j = i;j <= n;++j) {
            dijkstra(i,j);
        }
    }
    DP();
    printf("%d",f[1][n]);
    return 0;
}
```