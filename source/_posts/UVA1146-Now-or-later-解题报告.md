title: UVA1146 Now or later 解题报告
categories: 解题报告
tags: [图论,二分,2-SAT]
date: 2020-12-29 12:35:14
---

> [UVA1146 Now or later](https://www.luogu.com.cn/problem/UVA1146)
> 
> 有 $n$ 架飞机需要着陆。 每架飞机都可以选择“早着陆”和“晚着陆”两种方式之一，且必须选择一种。 第 $i$ 架飞机的早着陆时间为 $E_i$，晚着陆时间为 $L_i$，不得在其他时间着陆。 你的任务是为这些飞机安排着陆方式，使得相邻两个着陆时间间隔的最小值（称为安全间隔）应尽量大。
>
> $n \leq 2000,0 \leq t \leq 10 ^ 7$

<!--more-->

## 解题思路：

二分 + 2-SAT

首先分析一下题面，观察到**最小值最大**这个点就首先考虑二分。

二分时间间隔 $t$，接下来考虑如何判定在给定安全时间内是否有可行方案。

可以发现，对于一个飞机我们显然可以拆成两个点来处理，一个点表示早降落时间，一个点表示晚降落时间。

然后我们对于飞机两两考虑，我们记 $x_i$ 为第 $i$ 架飞机早到时间，$y_i$ 为第 $i$ 架飞机晚到时间，$x_j$ 为第 $j$ 架飞机早到时间，$y_j$ 为第 $j$ 架飞机早到时间，其中 $x_i > x_j$。

则如果 $x_i - x_j < t$，两个飞机肯定不能一起早着陆，也就是说，如果第 $i$ 架飞机早着陆，则第 $j$ 架飞机必须晚着陆，反之亦然。这里就出现了我们非常熟悉的**二元关系**，考虑使用 2-SAT 解决问题，在每次check的时候重新建图。

上面的约束条件转化为连边就是 $x_i \to y_j,y_i \to x_j$，这样子边的个数是 $n^2$ 级别的，加上二分，时间复杂度为 $O(n^2 \log t)$。

这题比较恶心的一点在于 UVA 原题卡了前向星，建议使用 `vector` （UVA 上是默认开 `-O2` 的），以及点数至少要开到两倍的 $n$

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
#include <stack>
using namespace std;
#define ll long long


const int N = 1e4 + 10;

int dfn[N],low[N],c[N],cnt,num,n;
bool ins[N];

stack <int> s;
vector <int> G[N];

void tarjan(int x) {
    dfn[x] = low[x] = ++cnt;
    s.push(x);ins[x] = 1;
    for (int i = 0;i < G[x].size();++i) {
        int v = G[x][i];
        if (!dfn[v]) {
            tarjan(v);
            low[x] = min(low[x],low[v]);
        } else if (ins[v]) {
            low[x] = min(low[x],dfn[v]);
        }
    }   
    if (dfn[x] == low[x]) {
        int y;c[x] = ++num;
        do {
            y = s.top();s.pop();
            c[y] = num;ins[y] = 0;
        } while (x != y);
    }
}

int a[N][2];

bool check(int cap) {
    for (int i = 1;i <= 2 * n;++i) G[i].clear();
    memset(dfn,0,sizeof dfn);
    memset(low,0,sizeof low);
    memset(c,0,sizeof c);
    memset(ins,0,sizeof ins);
    cnt = num = 0;
    for (int i = 1;i <= n;++i) {
        for (int qwq = 0;qwq <= 1;++qwq) {
            for (int j = i + 1;j <= n;++j) {
                for (int qaq = 0;qaq <= 1;++qaq) {
                    if (abs(a[i][qwq] - a[j][qaq]) < cap) {
                        G[i + n * qwq].push_back(j + n * (qaq ^ 1));
                        G[j + n * qaq].push_back(i + n * (qwq ^ 1));
                    }
                }
            }
        }
    }//重新建图的过程
    for (int i = 1;i <= 2 * n;++i) if (!dfn[i]) tarjan(i);
    for (int i = 1;i <= n;++i) {
        if (c[i] == c[i+n]) return 0;
    }
    return 1;
}

void solve() {
    memset(ins,0,sizeof ins);
    tot = 0;
    int l = 0,r = 0,ans = 0;
    for (int i = 1;i <= n;++i) scanf("%d %d\n",&a[i][0],&a[i][1]),r = max(r,a[i][1]);
    while (l < r - 1) {
        int mid = (l + r) >> 1;
        if (check(mid)) l = mid;
        else r = mid;
    }
    cout << l << endl;
    return ;
}

signed main() {
    while (scanf("%d\n",&n) != EOF) solve();
    return 0;
}
```