---\r\title: Luogu P3354 [IOI2005]Riv 河流 解题报告
categories: 解题报告
tags: [DP,树上问题,树形DP]
date: 2021-02-26 11:52:23
---

> [P3354 [IOI2005]Riv 河流](https://www.luogu.com.cn/problem/P3354)
> 
> Byteland 国，有 $n$ 个伐木的村庄，这些村庄都座落在河边。目前在 Bytetown，有一个巨大的伐木场，它处理着全国砍下的所有木料。木料被砍下后，顺着河流而被运到 Bytetown 的伐木场。
> 
> Byteland 的国王决定，为了减少运输木料的费用，再额外地建造 $k$ 个伐木场。这 $k$ 个伐木场将被建在其他村庄里。这些伐木场建造后，木料就不用都被送到 Bytetown 了，它们可以在运输过程中**第一个碰到的新伐木场**被处理。显然，如果伐木场座落的那个村子就不用再付运送木料的费用了。它们可以直接被本村的伐木场处理。
> 
> 注：所有的河流都不会分叉，形成一棵树，根结点是 Bytetown。
> 
> 国王的大臣计算出了每个村子每年要产多少木料，你的任务是决定在哪些村子建设伐木场能获得最小的运费。其中运费的计算方法为：每一吨木料每千米 $1$ 分钱。
> 
> $2\le n\le 100$，$1\le k\le \min(n,50)$

<!--more-->

Solution:

考虑如下的 DP：

设 $f_{x,k,las}$ 表示当前在结点 $x$，还可以建 $k$ 个伐木场，上一个伐木场是 $las$，并且当前点不建伐木场的最小代价.

$g_{x,k,las}$ 表示当前在结点 $x$，还可以建 $k$ 个伐木场，上一个伐木场是 $las$，并且当前点建伐木场的最小代价.

可以观察到，我们把 $k$ 这一维看做背包的体积维度，那么 $las$ 这一维就是我们转移用到的附加维度做一个树形背包。

那么接下来考虑转移：

1. 这个点不建伐木场，那么 $las$ ~~代代相传~~被下放到下面去。于是就有

$$
f_{x,k,las} = \sum_{y \in son(x)} \min_{0 \leq p \leq \min(k,siz_y)} (f_{x,k-p,las} + f_{y,p,las})
$$

2. 这个点建伐木场，那么对于儿子来说 $las$ 这维就变成了 $x$，并且 $x$ 这里的代价需要从 $g_{x,k-p,las}$ 中来。

$$
f_{x,k,las} = \sum_{y \in son(x)} \min_{0 \leq p \leq \min(k,siz_y)} (g_{x,k-p,las} + f_{y,p,x})
$$

最后，我们需要将每个节点的答案合并到 $f$ 上，这里的转移不难想，按照题目来走就行。

$$
\begin{aligned}
&k = 0:f_{x,k,las} = a_x * (dep_las -

 dep_x)\\
&k \not ={0}:f_{x,k,las} = \min(f_{x,k,las} + a_x * (dep_las - dep_x),g_{x,k-1,las})
\end{aligned}
$$

这题在代码实现的时候有比较多的细节，这里需要提一下：

1. 关于 $las$，代码里我用了一个 `vector` 来存并且遍历，注意从每个点走进来的时候要先把这个点丢进去，走出来的时候要把这个点丢出来，就像这样写：

```cpp
vector <int> anc;

void DP(int x) {
    anc.push_back(x);
    for (auto las:anc) {//这里运用了C++11的auto，如果不用C++11可以写迭代器/直接遍历下标
        //DP
    }
    anc.pop_back();
}
```

2. 关于树形背包的过程中，需要注意非法转移，也就是你在循环体积的时候要注意上界应该是 $\min(siz_x,k)$，同样在分组背包的过程中要注意上界应该是 $\min(siz_y,j)$.虽然这题实测你直接用 $k,j$ 来转移也没有啥问题，但是如果你做了 [P1273](https://www.luogu.com.cn/problem/P1273) 这个题，你就会意识到正确转移的重要性。

```cpp
for (int j = min(siz[x],k);j >= 0;--j) {
    for (int p = 1;p <= min(j,siz[y]);++p) {
    }
}
```

3. 关于树形背包的过程中，转移需要附上初值，这里就直接放代码。

```cpp
f[x][j][las] += f[y][0][las];
g[x][j][las] += f[y][0][x];
```

Code:

```cpp
const int N = 150;

int n,k,f[N][N][N],g[N][N][N],a[N],dep[N],siz[N];

vector <int> anc;
vector <pair<int,int> > G[N];

void predfs(int x) {
    siz[x] = 1;
    for (auto e:G[x]) {
        int y = e.first;
        dep[y] = dep[x] + e.second;
        predfs(y);
        siz[x] += siz[y];
    }
}

void DP(int x) {
    anc.push_back(x);
    for (auto e:G[x]) {
        int y = e.first;
        DP(y);
        for (auto las:anc) {
            for (int j = min(siz[x],k);j >= 0;--j) {
                f[x][j][las] += f[y][0][las];
                g[x][j][las] += f[y][0][x];
                for (int p = 1;p <= min(j,siz[y]);++p) {
                    f[x][j][las] = min(f[x][j][las],f[x][j-p][las] + f[y][p][las]);
                    g[x][j][las] = min(g[x][j][las],g[x][j-p][las] + f[y][p][x]); 
                }
            }
        }
    }

    for (auto las:anc) {
        for (int j = min(siz[x],k);j >= 1;--j) {
            f[x][j][las] = min(f[x][j][las] + a[x] * (dep[x] - dep[las]),g[x][j-1][las]);
        }
        f[x][0][las] += a[x] * (dep[x] - dep[las]);
    }

    anc.pop_back();
}

signed main() {
    read(n,k);
    for (int i = 1;i <= n;++i) {
        int x,w;
        read(a[i],x,w);
        G[x].push_back(make_pair(i,w));
    }
    predfs(0);
    DP(0);
    printf("%d\n",f[0][k][0]);
	return 0;
}
```