title:  ZROI 2022暑假AB班 Round6 解题报告
categories: 解题报告
tags: [莫比乌斯反演]
date: 2022-08-13 12:39:00
---
30 + 0 + 0，rk 38，暴力写挂。

<!--more-->

# A

首先我们考虑证明一个东西，即原问题有解当且仅当 $\gcd(x,y) | k,x + y \geq k$。

> 证明：
>
> 我们考虑构造一个操作序列来满足上述条件，设 $x< y$。
>
> 如果我们把 $x$ 倒 $k$ 桶进 $y$ 使得溢出，那么我们此时得到的就是 $kx\bmod y$，所以我们这里就可以构造出任何一个 $[0,y-1]$ 水量的桶，所以构造 $[x,x+y-1]$ 同样是简单的。

所以我们考虑就是要统计满足下列两个条件的 $(x,y)$ 对数：

- $\gcd(a_x,a_y) | k$
- $a_x + a_y \geq k$

$k = 1$ 的情况是 trivial 的，我们考虑就是求两个数互质，对式子莫反一下：
$$
\sum\limits_{x = 1}^n\sum\limits_{y = 1}^{x-1} [a_x+a_y\geq k] [\gcd(a_x,a_y) | 1]\\
= \sum\limits_{g | gcd(x,y)}\mu(g)\sum\limits_{x = 1}^n\sum\limits_{y = 1}^{x-1} [a_x+a_y\geq k]
$$
我们考虑每个点只会在因数的地方有贡献，所以贡献是 $n\log n$ 级别的。

对于 $k > 1$ 的情况，我们考虑从质因数的角度做一个容斥，减去一个质因子，加上两个质因子……

我们考虑对 $k$ 做一个唯一分解，$k = p_1^{r_1} \dots p_n^{r_n}$ 这样就变成对于第 $i$ 个质因子有一个 $\leq r_i$ 的约束。

所以我们可以强制某一个质因子不满足,对于一个 $i$，处理一个 $g_i$ 表示 $\gcd(a_x,a_y) = i,a_x + a_y \geq k$ 的方案数，枚举容斥项算一下，复杂度是调和级数 $\mathcal{O}(k\log k)$

这类题启发我们对值域进行莫比乌斯反演，这个套路就是比如求$\sum\limits_{i=1}^n\sum\limits_{j=1}^n [gcd(a_i,a_j)=1]$ 并且 $a$ 的值域不大的时候，

```cpp
const int N = 1e6 + 10;
int n,k,a[N],maxx;
ll ans;
ll mu[N], v[N],pri[N],tot,f[N];
ll x, cnt[N];
bool vis[N];
vector <int> tmp;
void init(int m) {
    mu[1] = 1;
    for (int i = 2;i <= m;++i) {
        if (!vis[i]) pri[++tot] = i,mu[i] = -1;
        for (int j = 1;j <= tot && pri[j] * i <= m;++j) {
            vis[pri[j] * i] = 1;
            if (i % pri[j] == 0) {
                break;
            }
            mu[i * pri[j]] = -mu[i];
        }
    }
}

bool cmp(int x,int y) {
    return x > y;
}

signed main() {
    //FO(p)
	read(n,k);
    for (int i = 1;i <= n;++i) read(a[i]),maxx = max(a[i],maxx),++cnt[a[i]];
    init(maxx);
    for (int i = 1;i <= maxx;++i) {
        int now = (maxx / i) * i + i,tmp = 0;
        //now = max(a_i) the most close | i
        for (int j = i;j <= maxx;j += i) {
            while (now && now + j >= k + i) {
                now -= i;
                tmp += cnt[now];
            }
            f[i] += tmp * cnt[j];
            if (now <= j) f[i] -= cnt[j];
        }
        f[i] /= 2;//chongfu pairs.
    }
    for (int i = 1;i <= k;++i) {
        if (k % i) continue;
        ll now = f[i];
        for (int j = i * 2;j <= maxx;j += i) {
            now += f[j] * mu[j / i];
        }
        ans += now;
    }
    printf("%lld\n",ans);
	return 0;
}

```

# B

## 笛卡尔树做法

我们考虑这个东西可以很显然转化为笛卡尔树上一个极长的 $m$ 的左子树内有 $l$，右子树内有 $r$，然后有多少 $a_l + a_r = X - a_m$ 的，这个东西可以 DSU on  tree 来做，复杂度是 $\mathcal{O}(n\log^2 n)$。

实现没写，参考这个老哥做法：[提交记录 #493594 - Zhengrui Online Judge (zhengruioi.com)](http://www.zhengruioi.com/submission/493594)

## 分治做法

我们直观考虑对 $[L,R]$ 分治，对于 $L \leq l \leq m \leq r \leq R$，我们可以讨论 $m \leq mid$ 和 $m > mid$ 的两种情况。

如果我们枚举这个 $l,r$，那么 $m$ 的取值就是显然的，我们记 $a_l$ 表示 $[l,mid]$ 的最大值，$b_r$ 表示 $[mid,r]$ 的最大值。

那么要满足：

- $a_l \geq b_r$
- $A_l + A_r = X - a_l$

那么假设枚举了 $l$ ，我们就得到了 $a_l$ 还有 $b_r$ 的一个上界，并且注意最大值个数是可变的，我们也要记录次数。

所以我们考虑这个 $r$ 肯定是一个前缀，只要做一次查询查出来 $r$ 位置集合大小就可以了。

分析一下复杂度，我们就是需要一个单点加，区间查询的数据结构，可以用数组实现（记得撤销保证复杂度）。右边的问题是对称的，一层分治的时间复杂度可以做到 $\mathcal{O}(n)$ ，总时间复杂度 $\mathcal{O}(n\log n)$

```cpp
const int N = 1e6 + 10;
int n,x,a[N],l[N],r[N];
ll ans;
ll buc[N];

void solve(int L,int R) {
    if (L == R) {
        if (a[L] * 3 == x) {
            ++ans;
        }
        return ;
    }
    int mid = (L + R) >> 1;
    ll maxx = 0,cnt = 0;
    int l,r;
    for (l = mid,r = mid + 1;l >= L;--l) {
        if (a[l] > maxx) maxx = a[l],cnt = 1;
        else if (a[l] == maxx) ++cnt;
        while (r <= R && a[r] <= maxx) buc[a[r]]++,++r;
        if (x >= a[l] + maxx) ans += cnt * buc[x - a[l] - maxx];
    }
    for (int i = mid + 1;i <= R;++i) buc[a[i]] = 0;
    maxx = cnt = 0;
    for (l = mid,r = mid + 1;r <= R;++r) {
        if (a[r] > maxx) maxx = a[r],cnt = 1;
        else if (a[r] == maxx) ++cnt;
        while (l >= L && a[l] <= maxx) buc[a[l]]++,--l;
        if (x >= a[r] + maxx) ans += cnt * buc[x - a[r] - maxx];
    }
    for (int i = L;i <= mid;++i) buc[a[i]] = 0;
    solve(L,mid);solve(mid + 1,R);
}

signed main() {
    read(n,x);
    for (int i = 1;i <= n;++i) read(a[i]);
    solve(1,n);
    printf("%lld\n",ans);
    return 0;
}
```