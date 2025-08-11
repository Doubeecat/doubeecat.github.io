title: ZROI 2022暑假AB班 Round1 解题报告
categories: 解题报告
tags: [DP,凸包,闵可夫斯基和,决策单调性]
date: 2022-08-08 13:19:00
---
60 + 0 + 0,rk 39

<!--more-->

# A

## 算法1

设计一个暴力 DP，令 $f_{i,k}$  表示前 $i$ 个元素被分成了 $k$ 段，$val(l,r)$ 表示 $[l,r]$ 这段元素的极差，可以对于每个区间用 ST 表预处理出来 $val$，然后直接转移：
$$
f_{i,k} = \max\limits_{k \leq j < i}\{f_{j,k - 1} + val_{j+1,i}\}
$$
时间复杂度 $\mathcal{O}(n^3)$

## 算法2

通过记录上方暴力 DP 的转移点，我们猜想这个 $val$ 满足决策单调性，事实上确实如此。

一个感性的证明是我们考虑多纳入一个元素考虑极差的时候，如果这个数不能取代当前的最大/最小值，那么极差不变，否则极差扩大。

所以我们可以直接使用 2D1D 的分治决策单调性板子，时间复杂度 $\mathcal{O}(n^2\log n)$

```cpp
ll f[N][N],st1[M][N],st2[M][N],trans[N][N];
void prework() {
    for (int i = 1;i <= n;++i) st1[0][i] = st2[0][i] = a[i];
    for (int i = 1;i <= 20;++i) {
        for (int j = 1;j + (1 << i) - 1 <= n;++j) {
            st1[i][j] = max(st1[i-1][j],st1[i-1][j + (1 << (i - 1))]);
            st2[i][j] = min(st2[i-1][j],st2[i-1][j + (1 << (i - 1))]);
        }
    }
}

int query(int typ,int l,int r) {
    int len = log(r - l + 1) / log(2);
    if (typ == 1) return max(st1[len][l],st1[len][r - (1 << len)+ 1]);
    else return min(st2[len][l],st2[len][r - (1 << len)+ 1]);
}

int val(int l,int r) {
    return query(1,l,r) - query(2,l,r);
}

void solve(int i,int l,int r,int L,int R) {
    if (l > r) return ;
    int pos = -1,mid = (l + r) >> 1;
    for (int j = max(i-1,L);j <= min(mid - 1,R);++j) {
        int w = f[j][i-1] + val(j+1,mid);
        if (w > f[mid][i]) {
            f[mid][i] = w;
            pos = j;trans[mid][i] = j;
        }
    }
    solve(i,l,mid - 1,L,pos);
    solve(i,mid + 1,r,pos,R);
}

void solver() {
    prework();
    for (int i = 1;i <= n;++i) solve(i,1,n,0,n-1);
    for (int i = 1;i <= n;++i) printf("%lld\n",f[n][i]);
}
```

## 算法3

在对拍的时候我们不难发现，最终的答案是具有凸性的，这点显然也可以感性理解（

正解的思路是，我们考虑构造一个序列 $b_1,\dots b_n$ 然后给最大值赋值 1，最小值赋值 -1，其他都是 0，求一个 $\max\{\sum a_i\times b_i\}$.

我们考虑分治，对于一个 $[l,r]$ ，我们考虑左侧是否有孤立的 1/-1,右边是否有孤立的 1/-1,一共分了 $k$ 段。 

然后我们发现可以合并两个区间，对于 $[l_1,r_1],[l_2,r_2]$ ，对于中间的两个只有 $1/-1$ 的段我们可以合并成一个段，也就转移到了新的状态 $[l_1,r_2]$，只需要判断 $[l_1,r_1]$ 右边和 $[l_2,r_2]$ 能不能拼成一段即可。

![image-20220808200529493](https://s2.loli.net/2022/08/08/kC8wVz5BPSEGy1u.png)

等于说我们设计状态 $f[l][r][k][0/1/2][0/1/2]$  $l,r,k$ 的状态定义相仿，第一个 $0/1/2$ 表示左端有 $1/-1$，第二个表示右端，$l,r$ 我们直接丢到分治的框架，然后分治内部就是处理每个 $k$ 了（这句有点意识流，感性理解，目的是为了说明分治如何替代 DP 过程）。

这个状态包含的其实是一个序列，代表 $[l,r]$ 这段的 DP 值，我们把合并的过程写出来就是：
$$
f[l_1][r_2][i][j] = \max\{f[l_1][r_1][k*2][i][1/2] + f[l_2][r_2][k*2+1][2/1][j] \}
$$
这个实际上就是一个 $(\max,+)$ 卷积的过程，并且因为这个东西是凸的，凸包的 $(\max,+)$ 卷积等于差分归并排序的前缀和这个实际上就是做了一个闵可夫斯基和，如图。

![img](https://cdn.luogu.com.cn/upload/image_hosting/saj70vi6.png)

所以我们可以做一个 two-pointer 来线性把两个状态的序列给合并。（这里实际上证明对于 $(\min,+)$ 卷积也是同理，我们对两个下凸壳是可以线性合并的捏）

那么算上分治，我们的总复杂度就变成了 $\mathcal{O}(n\log n)$。

```cpp
ll a[N],n;
vector <ll> f[N<<2][3][3];
//0->0 1->1 2->-1

vector <ll> merge(vector <ll> a,vector <ll> b) {//利用闵可夫斯基和合并两个凸包
    vector <ll> ans;
    /*for (auto x : a) printf("%d ",x);
    puts ("");
    
    for (auto x : b) printf("%d ",x);*/
    ll tmp = a[0] + b[0],p1 = 1,p2 = 1;
    ans.push_back(tmp);
    for (int i = a.size() - 1;i >= 1;--i) a[i] -= a[i-1];
    for (int i = b.size() - 1;i >= 1;--i) b[i] -= b[i-1];
    while (p1 < a.size() && p2 < b.size()) {
        if (a[p1] > b[p2]) tmp += a[p1],++p1;
        else tmp += b[p2],++p2;
        ans.push_back(tmp);
    }
    while (p1 < a.size()) tmp += a[p1],++p1,ans.push_back(tmp);
    while (p2 < b.size()) tmp += b[p2],++p2,ans.push_back(tmp);
    return ans;
}

void solve(int l,int r,int k) {
    for (int i = 0;i <= 2;++i) {
        for (int j = 0;j <= 2;++j) {
            f[k][i][j].resize(r - l + 2,-0x3f3f3f3f3f3f3f3f);
        }
    }

    if (l == r) {
        f[k][0][0][1] = 0;
        f[k][1][0][0] = f[k][0][1][0] = a[l];
        f[k][2][0][0] = f[k][0][2][0] = -a[l];
        return ;
    }

    int mid = (l + r) >> 1;
    solve(l,mid,k << 1);
    solve(mid + 1,r,k << 1 | 1);

    for (int i = 0;i <= 2;++i) {
        for (int j = 0;j <= 2;++j) {
            vector <ll> tmp = merge(f[k << 1][i][0],f[k << 1 | 1][0][j]);
            for (int p = 0;p <= r - l + 1;++p) f[k][i][j][p] = tmp[p];

            tmp = merge(f[k << 1][i][1],f[k << 1 | 1][2][j]);
            for (int p = 1;p <= r - l + 1;++p) f[k][i][j][p] = max(f[k][i][j][p],tmp[p-1]);

            tmp = merge(f[k << 1][i][2],f[k << 1 | 1][1][j]);
            for (int p = 1;p <= r - l + 1;++p) f[k][i][j][p] = max(f[k][i][j][p],tmp[p-1]);

            //合并凸包
            for (int p = 0;p <= mid - l + 1;++p) f[k][i][j][p] = max(f[k][i][j][p],f[k<<1][i][j][p]);
            for (int p = 0;p <= r-mid;++p) f[k][i][j][p] = max(f[k][i][j][p],f[k<<1|1][i][j][p]);
        }
    }
}


signed main() {
	read(n);
    for (int i = 1;i <= n;++i) read(a[i]);
    solve(1,n,1);
    for (int i = 1;i <= n;++i) printf("%lld\n",f[1][0][0][i]);
    return 0;
}
```

# B

> From llzer.

~~下次还填非常简单，考试的时候几乎想完了，可惜来不及学最小边覆盖，也写不来40pts的插头dp，遗憾离场~~

以下为考场想法：

对于每个格子上下左右四条边，接到管赋值为1，没接到管赋值为0，那么一个格子放了高级水管之后上下的边的权值异或和为1，左右的边的权值异或和也为1

定义**可爱的格子**为放了高级水管的格子，**不可爱的格子**为没放高级水管的格子

那么题目就是在合法的前提下最大化可爱的格子的数量

题目中给出的1234o的格子必须为可爱的格子，x必须为不可爱的格子，.能成为可爱的格子，也能成为不可爱的格子，~~反正就是随便整~~

发现1234ox其实是给出了一些限制，限制了边的权值，而1234已经确定了具体如何限制，ox则暂时并不能确定具体如何限制，只给出上下的边的权值异或和是否为1与左右的边的权值的异或和是否为1

观察一下就可以发现行和列基本上是独立的，我们可以分开来考虑

对于已经给出限制的边，我们考虑两条相邻的竖着的边，这两条边之间夹着的格子数大于等于2，手玩一下若是它们的权值的异或和为1，则这两条边之间夹着的所有格子中必然至少有一个不可爱的格子，且可以在选择一些格子为不可爱的格子后构造出剩下所有格子均为可爱的格子的方案，否则就可以全都是可爱的格子

列的话转90°就完事了

称**行限制**为对于两条竖着的已经给出限制的边，它们之间夹着的格子至少有一个为不可爱的格子，**列限制**为对于两条横着的已经给出限制的边，它们之间夹着的格子至少有一个为不可爱的格子

再手玩一下发现只对于一个行限制或是列限制来说，我们只要选择一个格子为不可爱的格子即可构造出剩下所有被夹着的格子均为可爱的格子的方案，01交错赋值即可，当然若是选择多个也可以

发现行限制与列限制之间可能会有一些交点，我是傻逼，写不来，而且有可能写出来的东西过几天我自己都不一定能看懂

日哦，换种表达

我们将这个网格图抽象成点之后扔到平面直角坐标系里，每个格子对应一个点

放在第一象限里，~~随便放哪都行，你就是横跨四个象限也没问题~~

将行限制定义为一条与x轴平行的线段，经过所有两条已经竖着的给出限制的边夹着的格子在平面直角坐标系中所对应的点，列限制定义为一条与y轴平行的线段，剩下的定义与前面类似

容易发现全部线段抽象出来后可能是会有交点的，如行限制$y=2(2<=x<=7)$与列限制$x=5(2<=y<=7)$很明显有一个交点(2,5)，那么我们可以选择(2,5)这个点在原网格图中对应的格子为不可爱的格子，这样就用一个格子满足了两个限制！~~我满足两个！~~~

再想一下，发现想到了图，然后把所有行限制扔到左边，列限制扔到右边，若一个行限制与一个列限制有一条边，就把它们连起来，选择这条边就等价于同时满足了这两个限制，称**孤立点**为在这张图里入度与出度均为0的边，然后发现选择的边数加上孤立点个数就是坏格子数量，我们只需要最小化选择的边数即可！

然后统计一下孤立点数量，拿掉孤立点，用剩下的二分图跑一遍最小边覆盖就行了！

~~然后我不会最小边覆盖，这真是个悲惨的故事~~

~~我的网络流水平跟我的多项式水平半斤八两，可能还多项式强一点，而且关于二分图我好像最大匹配都不会写了，真是太菜了~~

~~代码？等我学会最小边覆盖再说吧~~

# C

## 算法1

$\mathcal{O}(2^n \times n^2)$ 嗯造

写挂了，哈哈。

## 算法2

以下称国王所在点为 A 类点，人民所在点为 B 类点。

按照树形 DP 的套路，围绕子树设计状态。

考虑一个点 $x$ 的子树里含有的 A 类和 B 类点

- 对于 A 类点而言，我们只需要考虑深度最浅的，因为深度更深的 A 类点对于子树外的影响点集是深度最浅的影响点集的子集
- 对于 B 类点而言，我们考虑如果找到一个 A 类点对于深度最深的 B 类点满足条件，那么对于深度更浅的 B 类点也同样满足条件。

所以我们考虑 $f_{x,i,j}$ 为 $x$ 子树中最浅的一个 A 类点为 $i$，最深的一个没有找到对应 A 的 B 类点为 $j$ 此时的 $\max\{\sum\limits_{u \in subtree(x),u \in B} a_ut - \sum\limits_{u\in subtree(x),u \in A} s\}$ .

这样暴力做是 $\mathcal{O}(n^3)$ 的。

## 算法3

对于上边的两种情况，我们继续挖掘一下性质。

对于一个满足情况 2 的 B 类点，那么我们考虑如果存在一个深度最浅的 A 类点，他们的距离肯定是 $> d$ 的，那么子树外的 A 类点能覆盖的点集一定是深度最浅的 A 类点覆盖点的超集。因为我们考虑子树外存在的点 $A^{\prime}$ 到 B 类点 $\leq d$，也就是把深度最浅的 A 类点完爆了。

而对于一个满足情况 1 的 A 类点，我们的转移显然是和 B 类点没有什么关系的，所以直接省略。

所以我们把两类情况分开讨论，定义状态 $g_{x,i}$ 表示在 $x$ 的子树内最浅的 A 类点为 $i$，$f_{x,i}$ 表示 $x$ 子树内最深的一个没有找到对应 A 的 B 类点为 $i$，这个的转移需要分四种讨论如下（以下 $x,y$ 分别表示 $r$ 子树内的两个点）：

1. $x,y$ 均为 B 类点，将两个点的答案合并到深度较深的点上

   $g_{r,maxdep(x,y)} = g_{r,x} + g_{r,y}$  

2. $x$ 为 A 类点，$y$ 为 B 类点，讨论 $dis(x,y)$ 的情况，如果 $\leq d$ 就直接转移到 $x$ 上

   $f_{r,x} = f_{r,x} + g_{r,y}$

3. $x$ 为 B 类点，$y$ 为 A 类点，与上一种情况对称。

   $f_{r,y} = f_{r,y} + g_{r,x}$

4. $x,y$ 均为 A 类点，与第一种情况对称，但是合并到较浅的点上。

   $f_{r,mindep(x,y)} = f_{r,x} + f_{r,y}$

转移的时候注意利用 DFS 序转移，这样的复杂度才是正确的 $\mathcal{O}(n^2)$

```cpp
void DP(int x,int fa) {
    f[x][dfn[x]] = a[x] * t;
    g[x][dfn[x]] = a[x] * t - s;
    orz[x] = max(orz[x],g[x][dfn[x]]);
    siz[x] = 1;
    for (auto e : G[x]) {
        int y = e.first,w = e.second;
        if (y == fa) continue;
        DP(y,x);

        for (int i = dfn[x];i < dfn[x] + siz[x] + siz[y];++i) tmp1[i] = -0x3f3f3f3f;
        for (int i = dfn[x];i < dfn[x] + siz[x];++i) {
            for (int j = dfn[y];j < dfn[y] + siz[y];++j) {
                tmp1[mindep(i,j)] = max(tmp1[mindep(i,j)],g[x][i] + g[y][j]);
            }
        }
        
        for (int i = dfn[x];i < dfn[x] + siz[x] + siz[y];++i) tmp2[i] = tmp3[i] = -0x3f3f3f3f;
        for (int i = dfn[x];i < dfn[x] + siz[x];++i) {
            for (int j = dfn[y];j < dfn[y] + siz[y];++j) {
                if (dep[rk[i]] + dep[rk[j]] - 2 * dep[x] <= d) tmp2[j] = max(tmp2[j],f[x][i] + g[y][j]),tmp3[i] = max(tmp3[i],g[x][i] + f[y][j]);
            }
        }
        
        for (int i = dfn[x];i < dfn[x] + siz[x] + siz[y];++i) tmp4[i] = -0x3f3f3f3f;
        for (int i = dfn[x];i < dfn[x] + siz[x];++i) {
            for (int j = dfn[y];j < dfn[y] + siz[y];++j) {
                tmp4[maxdep(i,j)] = max(tmp4[maxdep(i,j)],f[x][i] + f[y][j]);
            }
        }
        for (int i = dfn[x] + siz[x];i < dfn[x] + siz[x] + siz[y];++i) g[x][i] = -0x3f3f3f3f;
        for (int i = dfn[x];i < dfn[x] + siz[x] + siz[y];++i) g[x][i] = max(g[x][i],max(max(tmp1[i],max(tmp2[i],tmp3[i])),g[x][i] + orz[y]));
        for (int i = dfn[x];i < dfn[x] + siz[x] + siz[y];++i) f[x][i] = max(f[x][i],max(tmp4[i],f[x][i] + orz[y]));
        orz[x] += orz[y]; siz[x]+= siz[y];

        for (int i = dfn[x];i < dfn[x] + siz[x];++i) orz[x] = max(orz[x],g[x][i]);
        for (int i = dfn[x];i < dfn[x] + siz[x];++i) f[x][i] = max(orz[x],f[x][i]);

    }
}
```