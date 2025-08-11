---
title: ZROI 2022NOIP10联测 Round 1 解题报告
categories: 解题报告
tags: [DP,思维,线段树,数学]
date: 2022-08-28 17:18:46
---
100 + 100 + 0 + 0,rk 42.

今天想不出骚话了。

<!--more-->

# A

假设最终选择的集合的平均数不超过 $k$。为使平均数不超过 $k$，应将 $\leq k$ 的数全部选入，然后贪心选择 $> k$ 的部分中最小的若干个数。因此从小到大排序后，最优解一定是一个前缀。

可以先将这个数组从小到大排序，然后依次比较每次去掉最大的一个元素之后的结果，符合条件所得的答案的最大值为所求。
因为如果每次去掉的是最小的数组元素的话，所得数组的平均值会变大，最后该数组中能满足严格大于这个数组的平均值的元素个数就会变小；反之，平均值会变小，结果就会有变大的可能性。所以每次比较要去掉最大的数。

这里通过二分实现，时间复杂度 $\mathcal{O}(n\log n)$

```cpp
const int N = 1e6 + 10;
ll n,a[N],sum,ans;

signed main() {
    //FO(ex_A2)
	read(n);
    for (int i = 1;i <= n;++i) read(a[i]);
    sort(a + 1,a + 1 + n);
    for (int i = 1;i <= n;++i) {
        sum += a[i];
        double now = sum / i;
        int l = 1,r = i;
        while (l < r) {
            int mid = (l + r + 1) >> 1;
            if (a[mid]-now <= eps) l = mid;
            else r = mid - 1;
        }
        ans = max(ans,(ll)i - r);
    }
    printf("%lld\n",ans);
	return 0;
}
```

# B

## 算法 1

我们考虑按照输入顺序对每条边考虑，将 $w_i = 0$ 称为非树边，$w_i = 1$ 称为树边。

那么对于每条树边我们直接赋值，对于每条非树边，我们考虑一个 Kruskal 的过程。$(x,y)$ 上每条边的边权都应该小于当前非树边边权。

所以我们暴力跳 LCA，然后暴力对每条树边赋值，时间复杂度 $\mathcal{O(n^2)}$。

## 算法 2

考虑优化算法 1，因为一条边最多被赋值一次，所以我们考虑使用并查集优化如上过程。我们把已赋权的边给合并一下，那么每次暴力跳 LCA 的过程就变成了 $\mathcal{O}(\log n)$ 的了。总时间复杂度 $\mathcal{O}(n\log n )$.

## 算法 3

提供一个不用什么脑子（虽然算法 2 也不用什么脑子）的树剖做法。

我们倒着考虑每条非树边，那么我们只需要对树上 $(x,y)$ 这条路径上所有边打一个 tag 表示它的赋值必须 $\leq tag$，然后最后对每条边查询一下 `tag`，用一个 `set` 之类的东西维护一下就行。（虽然考场上写了个玄学东西）

时间复杂度 $\mathcal{O}(n\log^2 n)$，幸运的是跑的飞快，过了。

```cpp
int m,dep[N],siz[N],top[N],fat[N],son[N],dfn[N],rk[N],cnt,a[N],num[N],to[N],p[N],f[N],ans[N];

vector <int> G[N];

void dfs1(int x,int f) {
    dep[x] = dep[f] + 1;
    siz[x] = 1;
    fat[x] = f;
    son[x] = 0;
    for (auto y : G[x]) {
        if (y != f) {

            dfs1(y,x);
            siz[x] += siz[y];
            if (siz[y] > siz[son[x]]) {
                son[x] = y;
            }
        }
    }
}

void dfs2(int x,int f) {
    dfn[x] = ++cnt;rk[cnt] = x;
    if (son[f] == x) top[x] = top[f];
    else top[x] = x;
    if (son[x]) {
        dfs2(son[x],x);
    }
    for (auto y : G[x]) {
        if (y != f && y != son[x]) {
            dfs2(y,x);
        }
    }
}

struct node {
    int l,r;
    int val,tag;
}tree[N<<2];

void build(int p,int l,int r) {
    tree[p].l = l,tree[p].r = r;
    if (l == r) {
        return ;
    }
    int mid = l+r >> 1;
    build(p << 1,l,mid);
    build(p << 1 | 1,mid + 1,r);
}

void pushdown(int p) {
    if (tree[p].tag) {
        tree[p << 1].tag = tree[p << 1 | 1].tag = tree[p].tag;
        tree[p << 1].val = tree[p << 1 | 1].val = tree[p].tag;
        tree[p].tag = 0;
    }
}

void change(int p,int x,int y,int k) {
    if (tree[p].l >= x && tree[p].r <= y) {
        tree[p].val = k;
        tree[p].tag = k;
        return ;
    }
    pushdown(p);
    int mid = (tree[p].l + tree[p].r) >> 1;
    if (x <= mid) change(p << 1,x,y,k);
    if (y > mid) change(p << 1 | 1,x,y,k);
}

int query(int p,int x) {
    if (tree[p].l == tree[p].r) {
        return tree[p].val;
    }
    pushdown(p);
    int mid = (tree[p].l + tree[p].r) >> 1,ans = 0;
    if (x <= mid) return query(p << 1,x);
    else return query(p << 1 | 1,x);
}

void changeline(int x,int y,int k) {
    while (top[x] != top[y]) {
        if (dep[top[x]] < dep[top[y]]) swap(x,y);
        change(1,dfn[top[x]],dfn[x],k);
        x = fat[top[x]];
    }
    if (dep[x] > dep[y]) swap(x,y);
    change(1,dfn[x] + 1,dfn[y],k);
    return ;
}

int querytree(int x) {
    return query(1,dfn[x]);
}

vector <pair<int,pii > > e1,e;

void main() {
    read(m);
    for (int i = 1;i <= m;++i) {
        int x,y,w;read(x,y,w);
        if (w) {
            G[x].push_back(y);
            G[y].push_back(x);
            e1.push_back(mp(i,mp(x,y)));
        } else {
            e.push_back(mp(i,mp(x,y)));
        }
    }
    dfs1(1,0);
    dfs2(1,0);
    build(1,1,n);
    for (auto ed : e1) {
        int x = ed.second.first,y = ed.second.second;
        if (dep[x] < dep[y]) swap(x,y);
        to[ed.first] = x;
    } 
    reverse(e.begin(),e.end());
    for(auto ed : e) {
        int x = ed.second.first,y = ed.second.second,k = ed.first;
        changeline(x,y,k);
    }
    reverse(e.begin(),e.end());
    for (auto ed : e1) {
        ans[ed.first] = min(ed.first,querytree(to[ed.first]));
        if (!ans[ed.first]) ans[ed.first] = ed.first;
    }
    for (auto ed : e1) {
        f[ed.first] = f[ed.first] + p[ans[ed.first]] + 1;p[ans[ed.first]]++;
    }
    for (auto ed : e) {
        p[ed.first]++;
    }
    for (int i = 1;i <= m;++i) p[i] += p[i-1];
    for (auto ed : e1) {
        f[ed.first] = f[ed.first] + p[ans[ed.first]-1];
    }
    for (auto ed : e) {
        f[ed.first] = p[ed.first];
    }
    for (int i = 1;i <= m;++i) printf("%d ",f[i]);
}
```

# C

## 算法 1

首先我们需要观察出最重要的一个性质：最后肯定是删除经过某些位置的所有区间。所以我们直接考虑从值域的角度来思考：

我们设计一个 DP，令 $f_i$ 表示考虑以 $i$ 为分界点的一个联通块，我们考虑转移。对于上一个转移点 $j$，我们显然只需要保留 $[j+1,i]$ 内最大的 $k$ 个区间即可。转移方程简单写出：
$$
f_i = \min\limits_{j \leq i}\{ f_j + val_{i+1,j}\}
$$
暴力转移是 $\mathcal{O}(n^3\log n)$ 的。

## 算法 2

我们对于上文提到的转移方法进行优化，从大到小枚举 $j$，用一个优先队列维护权值，这样就把计算 $val$ 的过程变成了单 log 的。

时间复杂度 $\mathcal{O}(n^2 \log n)$.

```cpp
const int N = 5e3 + 10;
int n,k,a[N],vis[N],cnt,ans,f[N];
pair<int,pii > p[N];
vector <int> buc[N],b;

bool cmp(pair<int,pii > a,pair<int,pii > b) {
    return a.second.second < b.second.second;
}

signed main() {
    read(n,k);
    for (int i = 1;i <= n;++i) {
        read(p[i].second.first,p[i].second.second,p[i].first);
        b.push_back(p[i].second.first),b.push_back(p[i].second.second);
        ans += p[i].first;
    }
    sort(b.begin(),b.end());
    b.erase(unique(b.begin(),b.end()) ,b.end());
    for (int i = 1;i <= n;++i) {
        p[i].second.first = lower_bound(b.begin(),b.end(),p[i].second.first)-b.begin()+1;
        p[i].second.second = lower_bound(b.begin(),b.end(),p[i].second.second)-b.begin()+1;
    }
    sort(p+1,p+1+n,cmp);
    int m = b.size(),pos = 1;
    for (int i = 1;i <= m;++i) {
        while (pos <= n && p[pos].second.second <= i) {
            int x = p[pos].second.first,y = p[pos].second.second,w = p[pos].first;
            buc[x].push_back(w);++pos;
        }
        priority_queue <int> q;
            int cnt = 0,sum = 0;
        for (int j = i - 1;j >= 0;--j) {
            for (auto e : buc[j+1]) {
                if (cnt < k) {
                    ++cnt;q.push(-e);
                    sum += e;
                } else {
                    int tmp = -q.top();
                    if (e > tmp) q.pop(),q.push(-e),sum += e-tmp;
                }
            }
            f[i] = max(f[i],f[j] + sum);
        }
        //printf("%d ",f[i]);
    }
    printf("%lld\n",ans - f[m]);
}
```

