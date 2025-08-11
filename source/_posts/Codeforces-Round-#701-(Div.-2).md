---
title: Codeforces Round #701 (Div. 2)
categories: Codeforces赛后题解
tags: [DP,暴力,树形DP,构造,数学,数论分块]
date: 2021-02-14 16:12:33
---

咕咕咕了两天，过春节去了

赛时通过：A B C ，总 rank $1629$

<!--more-->

# [A. Add and Divide](https://codeforces.ml/contest/1485/problem/A)
> 
> 有两个数 $a$ 和 $b$，每次可以选择把 $a$ 除去 $b$ （下取整）或者把 $b$ 加 $1$ ，问最少操作几次使 $a$ 等于 $0$。
> 
> $a,b \leq 10^9$

## Solution：

首先可以发现，这个次数不会很大，最多不超过 $32$ 次。因为 $2^{31} = 2147483648 > 10^9$。

其次我们发现，第二个操作肯定先放在前面，所以我们可以暴搜（雾）

## Code：

```cpp
int judge(int p,int q) {
    int ans = 0;
    while (p) {
        p /= q;
        ans++;
    }
    return ans;
}

void solve() {
    ll a,b;cin >> a >> b;
    int ans = 0;
    while (1) {
        //printf("%d %d\n",a,b);
        if (b <= 1) {
            ++b;
            ++ans;
            continue; 
        } else if (judge(a,b) >= judge(a,b+1) + 1) {
            ++b;
            ++ans;
            continue;
        }
        ans += judge(a,b);
        break;
    }
    printf("%d\n",ans);
}
```

# [B. Replace and Keep Sorted](https://codeforces.ml/contest/1485/problem/B)
> 
> 定义 $b$ 与 $a$ 相似当且仅当：
> 
> $a,b$ 为**严格单调递增**的**等长**正数序列
> 
> $a,b$ 中的每个元素都在 $[1,k]$ 中，$a,b$只有一个元素不同。
> 
> 有 $q$ 个询问，每次询问与 $a$ 的一个子段相似的 $b$ 有多少个。
> 
> $n,q \leq 10^5$

## Solution:

注意到，两个相似的序列中只有一个数不同，所以我们对于每个数依次考虑，直接利用加法原理统计答案即可。

那么考虑每一位怎么统计答案，比如 `2 4 7` 我们考虑中间 $4$ 这位。因为是严格单调递增，所以可取的值显然是 $3,5,6$ （注意要去掉自己），我们可以推导出公式，设 $f_i$ 为第 $i$ 位修改方案数，则有 $f_i = a_{i+1} - a{i-1} - 2$，这个东西可以前缀和优化掉。

但是注意一点，每一个序列的最开始那个数的取值范围是 $[1,a_2)$，最后一个数的取值范围是 $(a_{len} - 1,k]$,最后得到的式子就是 $ans = pre_{r-1} - pre_{l} + a_{l+1} - 2 + k - a_{r-1} -1$

## Code:

```cpp
signed main() {
	read(n,m,q);
    for (int i = 1;i <= n;++i) {
        read(a[i]);
    }
    a[0] = 0;a[n+1] = q;
    pre[1] = ans[1] = a[2] - 2;
    for (int i = 2;i <= n;++i) {
        ans[i] = (a[i+1]-1) - (a[i-1]+1);
        pre[i] = pre[i-1] + ans[i]; 
        //printf("%d %d\n",ans[i],pre[i]);
    }
    while (m--) {
        int l,r;read(l,r);
        if (l == r) {
            printf("%d\n",q-1);
        }
        else {
            printf("%d\n",pre[r-1] - pre[l] + a[l+1] - 2 + q - a[r-1] -1);
            //printf("%d\n",pre[r] - pre[l] + a[l]-1);
        }
    }   
	return 0;
}
```

# [C. Floor and Mod](https://codeforces.ml/contest/1485/problem/C)
> 
> 定义一对数 $(a,b)$是漂亮的，有 $\lfloor\frac{a}{b}\rfloor = a \mod b$，求 $1\le a \le x, 1 \le b \le y$ 中有多少对 $(a,b)\$是漂亮的。
> 
> $x,y \leq 10^9$

## Solution:

首先我们把式子化简一下：

$$
\begin{aligned}
    \lfloor\frac{a}{b}\rfloor &= a \mod b\\
    \lfloor\frac{a}{b}\rfloor &= a - (\lfloor\frac{a}{b}\rfloor * b)\\
    (b+1) \lfloor\frac{a}{b}\rfloor &= a \\
    \lfloor\frac{a}{b}\rfloor &= \frac{a}{b+1} = k\\
\end{aligned}
$$

得 $a = kb + k(k < b)$ ，并且注意到，如果没有限制的话，实际上会有 $b-1$ 组解。

所以我们可以枚举这个 $b$ ，得
$$
\sum_{i=2}^y \min(i-1,\frac{x}{i+1})
$$

然后我们把这个 $\min$ 拆开，找到中间点 $i = \sqrt{x+1}$，最后得到这个式子

$$
\sum_{i=2}^{\sqrt{x+1}} i-1 + \sum_{i=\sqrt{x+1}+1}^y \frac{x}{i+1}
$$

前面暴力计算即可，后面数论分块。

但是这边的数论分块形式并不像我们平常见到的 $\sum_{i=1}^n \frac{x}{i}$，所以我们可以稍加处理一下代码，变成：

```cpp
for (;l <= x && l-1 <= y; l = r + 1) {
    r = x/(x/l);
        if (r <= y + 1) ans += (r - l + 1) * (x / l);
        else break;
    }
    if (l <= x) {
        r = y + 1;ans += (r - l + 1) * (x / l);
}
```

感觉这题是本场除了 EF 以外的最难题，代码难写，式子难推，之后还要加强这块训练。
时间复杂度 $O(\sqrt{n})$

## Code:
```cpp
void solve() {
    ll l,r;ans = 0;
    read(x,y);
    /*for (ll i = 1;i <= y;++i) {
        r = min(i * i + i - 1,x);
        ans += r / (i+1);
    }*/
    for (l = 1;l * l + l - 1 <= x && l <= y;++l) {
        ans += (l * l + l - 1) / (l+1);
    }
    ++l;
    for (;l <= x && l-1 <= y; l = r + 1) {
        r = x/(x/l);
        if (r <= y + 1) ans += (r - l + 1) * (x / l);
        else break;
    }
    if (l <= x) {
        r = y + 1;ans += (r - l + 1) * (x / l);
    }

    printf("%lld\n",ans);
}
```

# [D. Multiples and Power Differences](https://codeforces.ml/contest/1485/problem/D)
> 
> 有一个 $n\times m$的矩阵 $a$
> 
> 要求构造一个矩阵 $b$ 使得 $b$ 中的每个元素都是 $a$ 中对应元素的倍数
> 
> 并且 $b$ 矩阵每个元素与上下左右四个元素之间的差的绝对值是某个整数的四次方。
> $n,m \leq 500,a_{i,j} \leq 16$

## Solution:

良心构造题。

这题有个比较牛逼的做法，首先我们考虑 $1\dots 16$ 的最小公倍数为 $720720$，然后对整个矩阵黑白染色，如果当前是黑色，那么就是 $720720$。如果当前是白色，那么就是 $720720 + a_{i,j} ^ 4$，证明显然。

时间复杂度 $O(nm)$

## Code:
```cpp
signed main() {
	read(n,m);
    for (int i = 1;i <= n;++i) {
        for (int j = 1;j <= m;++j) {
            read(a[i][j]);
            if (i + j & 1) printf("720720 ");
            else printf("%d ",720720-a[i][j]*a[i][j]*a[i][j]*a[i][j]);
        }
        puts("");
    }
	return 0;
}
```

# [E. Move and Swap](https://codeforces.ml/contest/1485/problem/E)
> 
> 有一棵根为 $1$ 的树，根到所有叶子距离相等。
> 
> 一开始红和蓝都在 $1$ 号点，每次红可以选择一个儿子移动下去，蓝可以任意移动到下一层的一个节点。
> 
> 然后你可以选择交换或者不交换红和蓝的位置，对答案产生 $|a_r - a_b|$ 的贡献。
> 
> 问把红和蓝最终移动到叶子上是总贡献最大是多少。
> 
> $n \leq 2 \times10^5$

## Solution:

首先我们肯定把点按照到 $1$ 的距离分成若干组，然后考虑怎么计算这个过程。

我们先定义 $f_i$ 为每一次操作完（也就是不论交不交换之后的过程）当红在点 $i$ 时的最大答案。

那么我们可以分两种情况讨论：

1. 没有交换，红直接走下来

    那么蓝点就比较好考虑了，我们直接走当前层的最大/最小值，让差最大即可。

2. 交换了

    这个就相当于，我红可以随便跑了，变成蓝找一个点走下去。方程为
    $f_x = \max{(f_{fa_x} + |a_y - a_x|)}$ 

    我们考虑把后面的绝对值拆开，得到方程 $f_x = \max{(f_{fa_x} + \max{(a_y - a_x,a_x - a_y)})}$ 

    那么你注意到 $a_x$ 实际上是一个定值，我们对于每一层先预处理出 $f_{fa_x} + a_y$ 和 $f_{fa_x} - a_y$ 的最大值即可。

这个方程其实并不难想，主要是要对于整个题目要有一定的思考，时间复杂度 $O(n)$

## Code:

```cpp
vector <int> son[N],grou[N];

ll dep[N],maxx[N],minn[N],n,a[N],maxdep,fa[N],ans,f[N];

void dfs(int x) {
    //printf("%d\n",x);
    for (auto y:son[x]) {
        dep[y] = dep[x] + 1;   
        maxdep = max(maxdep,dep[y]);
        grou[dep[y]].push_back(y);
        maxx[dep[y]] = max(maxx[dep[y]],a[y]);
        minn[dep[y]] = min(minn[dep[y]],a[y]);
        dfs(y);
    }
}

void solve() {
    read(n);
    for (int i = 1;i <= n;++i) son[i].clear(),grou[i].clear(),fa[i] = 0;

    for (int i = 2;i <= n;++i) {
        int x;read(x);son[x].push_back(i);
        fa[i] = x;
    }
    for (int i = 1;i <= n;++i) maxx[i] = 0,minn[i] = 0x3f3f3f3f,maxdep = ans = 0,f[i] = 0;
    for (int i = 2;i <= n;++i) {
        read(a[i]);
    }
    dep[1] = 1;
    dfs(1);
    for (int i = 2;i <= maxdep;++i) {
        ll maxa = -0x3f3f3f3f,maxb = -0x3f3f3f3f;
        for (auto x:grou[i]) {
            maxa = max(maxa,f[fa[x]] + a[x]);
            maxb = max(maxb,f[fa[x]] - a[x]);
        }//红点直接从父亲走下来
        for (auto x:grou[i]) {
            f[x] = max(f[x],f[fa[x]] + max(a[x] - minn[i],maxx[i] - a[x]));
            //直接从父亲走下来
            f[x] = max(f[x],max(maxa - a[x],maxb + a[x]));
            //走下来之后交换位置
            ans = max(ans,f[x]);
        }
    }
    printf("%lld\n",ans);
}
```

# [F. Copy or Prefix Sum](https://codeforces.ml/contest/1485/problem/F)
> 
> 有一数列 $b$，要求构造数列 $a$，使得每个 $b_i$ 满足 
> - $b_i=a_i$ 或 
> 
> - $b_i = \sum_{k=1}^{i}a_k$ 
> 
> 求构造方案数。
> 
> $n \leq 2 \times 10^5,-10^9 \leq b_i \leq 10^9$

## Solution:

首先思考下为什么答案不是 $2^n$，因为当 $\sum_{k=1}^{i-1}a_k = 0$ 时你会多算一遍。

所以我们先考虑一个比较简单粗暴的 DP，令 $f_{i,j}$ 表示我们当前构造到第 $i$ 个数，前缀和为 $j$ 时的方案数，转移依题意有两种：

- $f_{i,j+b_i} = \sum f_{i-1,j}$ 其中 $a_i = b_i$
- $f_{i,b_i} = \sum f_{i-1,j}$ 其中 $b_i = a_i + \sum_{k=1}^{i-1}a_k$

记得减去第一种 $j = 0$ 的情况。

我们考虑一下怎么去掉第一维：

- 第一种转移，实际上就是将整个 $f$ 数组平移了 $b_i$，这个我们可以通过一个 $delta$ 来解决。
- 第二种转移，实际上就是 $f_{b_i}$ 加上了整个数组的和，但当 $j = 0$ 时重复计算了，而在第一种转移的时候只是让 $f_{b_i}$ 位置变成了 $f_0$ ，所以直接让第二个位置等于所有元素的和就行了。
- 统计全局和，实际上除了 $b_i$ 这个位置全局和变了以外其他都没变，所以这边就直接 $sum = sum \times 2 - b_i$

注意这边 $b_i$ 的范围很抽象，所以我们开个 map 存 $f$ 数组即可。

时间复杂度 $O(n \log n)$

## Code:

```cpp
map <ll,ll> f;

ll n,a[N];

void solve() {
    f.clear();
    read(n);
    for (ll i = 1;i <= n;++i) {
        read(a[i]);
    }
    ll ans = 1,sum = 0;
    f[0] = 1;
    for (ll i = 1;i <= n;++i) {
        ll tmp = f[sum];f[sum] = ans;sum -= a[i];
        ans = (ans + ans - tmp + mod) % mod;
    }
    printf("%lld\n",ans);
}
```