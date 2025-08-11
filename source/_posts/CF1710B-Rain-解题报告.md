title: CF1710B Rain 解题报告
categories: 解题报告
tags: [思维]
date: 2022-07-26 17:52:00
---
> 有 $n$ 天在下雨，每一天，雨会在第 $x_i$ 个地方降落，降雨量为 $p_i$。降雨量会累加，对于一个地方 $j$，它的总降雨量为 $a_j$，每一次降雨其能接收到的降雨量为 $\max(0,p_i-|x_i-j|)$ 。
>
> 一个地方为**发大水**的定义为：在任何时间段有 $a_j > m$ .
>
> 你需要求出：对于每一天，独立地将该天的降雨量撤销之后，这一天是否还有地方是发大水的。
>
> $n \leq 2\times 10^5,p_i \leq 10^9,m \leq 10^9$

<!--more-->

以下称第 $i$ 天降雨的位置 $x_i$ 为关键点 $i$。

首先观察到一个性质：我们只需要对每个降雨的 $x_i$ 讨论是否发大水。

> 简略证明：
>
> 我们可以通过讨论相邻的两个关键点 $x_i,x_j (x_i < x_j)$ ，降雨量分别为 $p_i,p_j$ ，假设他们中间隔着一个点 $pos$.
>
> 可以得出，
> $$
> a_{x_i} = p_i + p_j - (x_j - x_i), \\
> a_{pos} = p_i + p_j - (pos - x_i + x_j - pos) = p_i + p_j - (x_j - x_i) = a_{x_i},\\
> a_{y_i} = p_j + p_i - (x_j - x_i)
> $$
> 这个实际上证明了，对于两个关键点之间的所有点受到的降雨量都是相同的。
>
> 故我们只需要讨论每个关键点即可。

接下来，每次降雨实际上可以看成加一个起点为 $x_i - p_i$，首项为 $0$，公差为 $1$，项数为 $p_i - x_i$ 的 等差数列，以及一个起点为 $x_i + 1$ ，首项为 $p_i$，公差为 $-1$，项数同样为 $p_i - x_i$ 的等差数列。

接下来我们发现，去掉一个降雨的地方其实是很复杂的，如果要强行上线段树的话会相对复杂，这里我们不妨转化一下。

我们考虑对于一个关键点 $x$，如果去掉第 $i$ 天的降雨可以让其变为不发大水的当且仅当 $p_i-|x_i-j| \geq a_j-m$，这个的证明直接展开即可。

接下来我们考虑把上面的几个东西综合一下，做一个扫描线。我们按照上面所说把每个降雨过程拆成两条线段（其实就像官方题解，是一个 $k = \pm 1$ 的斜线），再把每个关键点排序，考虑每次把关键点之前的直线都计算对关键点的影响。这是可以通过上面的计算式做到的。（这段不是很理解可以看代码）

这样我们就计算出了 $a_i$，最后判断一下即可。

时间复杂度为 $\mathcal{O}(n\log n)$，这个题总的来说其实出的非常 OI，一步一步发现性质，抽丝剥茧，个人认为虽然这场 Div.1 办的非常失败，但是这个题是个好题。

```cpp
//Every night that summer just to seal my fate
//And I scream, "For whatever it's worth
//I love you, ain't that the worst thing you ever heard?"
#pragma GCC optimize(2)
#include <algorithm>
#include <iostream>
#include <cstring>
#include <cstdio>
#include <cctype>
#include <vector>
#include <cmath>
#include <unordered_map>
using namespace std;
#define pii pair<int,int>
#define mp make_pair
#define ll long long
#define int long long

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

const int N = 2e5 + 10;

struct node {
    int id,pos,cnt;
    friend inline bool operator < (const node &a,const node &b) {
        return a.pos < b.pos;
    };
}q[N];

struct que {
    int pos,cnt,opt,flag;
    friend inline bool operator < (const que &a,const que &b) {
        return a.pos < b.pos;
    };
}l[N<<3];

int n,m,x[N],pos[N],pre[N],suf[N],tot = 0;
unordered_map <int,int> ans;

void add(int i) {
    l[++tot] = (que){x[i] - pos[i],pos[i] - x[i],1,1};
    //上升端的一段，位置可以延伸到x[i] - pos[i]，题目中绝对值拆一部分是pos[i] - x[i]
    l[++tot] = (que){x[i] + 1,x[i] + pos[i],0,1};
    l[++tot] = (que){x[i] + 1,- x[i] + pos[i],1,0};
    //转变一下上升下降 两个最终会相加得到2*x[i]
    l[++tot] = (que){x[i] + pos[i] + 1,x[i] + pos[i],0,0};
    //下降
}

void solve() {
    read(n,m);ans.clear();tot = 0;
    for (int i = 1;i <= n;++i) {
        read(x[i],pos[i]);
        q[i] = (node){i,x[i],0};
        add(i);
    }
    sort(q+1,q+1+n);sort(l+1,l+1+tot);
    int now = 1;
    int c1 = 0,c2 = 0,s1 = 0,s2 = 0;
    for (int i = 1;i <= n;++i) {
        while (now <= tot && l[now].pos <= q[i].pos) {
            if (l[now].opt) {
                if (l[now].flag) ++c1,s1 += l[now].cnt;
                else --c1,s1 -= l[now].cnt;
            } else {
                if (l[now].flag) ++c2,s2 += l[now].cnt;
                else --c2,s2 -= l[now].cnt;
            }
            ++now;
        }
        q[i].cnt = q[i].pos;
        q[i].cnt *= (c1 - c2);
        q[i].cnt += s1 + s2;
    }
    //q[i].cnt 求出最大覆盖
    for (int i = 1;i <= n;++i) {
        if (q[i].cnt > m) {
            suf[i] = (q[i].cnt - m) + q[i].pos;
            pre[i] = (q[i].cnt - m) - q[i].pos;
        } else {
            pre[i] = suf[i] = -1e18;
        }
    }
            pre[0] = suf[n+1] = -1e18;
    for (int i = 1;i <= n;++i) pre[i] = max(pre[i],pre[i-1]);
    for (int i = n;i >= 1;--i) suf[i] = max(suf[i],suf[i+1]);
    for (int i = 1;i <= n;++i) {
        ans[q[i].pos] = max(ans[q[i].pos],
                max(suf[i] - q[i].pos,pre[i] + q[i].pos));
    }
    for (int i = 1;i <= n;++i) {
        if (pos[i] >= ans[x[i]]) putchar('1');
        else putchar('0');
    }
    puts("");
}

signed main() {
    int T;read(T);
    while (T--) solve();
	return 0;
}

```

