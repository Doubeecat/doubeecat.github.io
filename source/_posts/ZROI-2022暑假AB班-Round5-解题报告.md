title: ZROI 2022暑假AB班 Round5 解题报告
categories: 解题报告
tags: [容斥原理,根号分治,分治NTT,NTT,多项式]
date: 2022-08-12 14:49:00
---
20+0+5,rk 51.

遗言丁真，鉴定为正解被卡。

<!--more-->

# A

对于这种怎么弄都没啥思路的题，可以先考虑根号分治一手。

我们考虑取一个分界点 $T$：

- 对于两个 $l < T$ 的向量，我们考虑直接暴力相乘时间复杂度是 $\mathcal{O}(T^2)$ 的，看起来就很对！所以我们直接先放着。
- 对于一个 $l_x >T$,一个 $l_y < T$ 的，我们考虑对于每个循环节预处理出长向量的和，这样我们就能在  $\mathcal{O}(l_y) \leq O(T)$ 的时间里求出来。
- 对于两个都 $l > T$ 的向量，我们考虑这样的询问最多会出现 $O(\frac{n}{T})$ 次，那么对于这部分我们还是暴力相乘，复杂度最多会达到 $\mathcal{O}(\frac{n^2}{T})$

那么我们考虑对 $T$ 取一个 $\sqrt{n}$ 的值，并且我们做一个记忆化，这样就保证了我们的复杂度是 $\mathcal{O}(n\sqrt n)$ 的，可以通过本题。

```cpp
const int N = 2e5 + 10;
int q,l[N],summ,T,pos[N],tot;
int sum[500][500][500];
vector <ll> v[N];
map <pii,ll> ans;

signed main() {
    read(n);
    for (int i = 1;i <= n;++i) {
        read(l[i]);summ += l[i];
        v[i].resize(l[i]);
        for (int j = 0;j < l[i];++j) {
            read(v[i][j]);
        }
    }
    int T = sqrt(summ);
    for (int i = 1;i <= n;++i) {
        if (l[i] > T) {
            int now = pos[i] = ++tot;
            for (int j = 1;j <= T;++j) {
                for (int k = 0,p = 0;k < l[i];++k,++p) {
                    if (p == j) p = 0;
                    sum[now][j][p] = (sum[now][j][p] + v[i][k]) % mod;
                }
            }
        }
    }
    read(q);
    for (int i = 1;i <= q;++i) {
        int x,y;read(x,y);
        if (l[x] > l[y]) swap(x,y);
        if (ans.count(mp(x,y))) {
            printf("%lld\n",ans[mp(x,y)]);
            continue;
        }
        ll tmp = 0;
        if (l[x] <= T && l[y] > T) {
            int p = pos[y];
            for (int i = 0;i < l[x];++i) {
                tmp = (tmp + v[x][i] * sum[p][l[x]][i] % mod) % mod;  
            }
        } else {
            for (int k = 0,p = 0;k < l[y];++k,++p) {
                if (p == l[x]) p = 0;
                tmp = (tmp + v[y][k] * v[x][p] % mod) % mod;
            }
        }
        printf("%lld\n",ans[mp(x,y)] = tmp);
    }
    return 0;
}
```

# B

本题启示：对于一个角度容斥不通的话尝试对另外一个角度进行容斥，如本题的球和桶，对桶容斥就是困难的，但是考虑球就简单很多。

## 算法 1

考虑对问题容斥，限制有 $m$ 个即每个位置不能放某种颜色的球。 

$f_{i,k}$表示当前到第 $i$ 个桶，强制不满足的限制有 $k$ 条的贡献。 则转移为 
$$
f_{i,k} = \sum\limits_{0\leq j \leq a_i} f_{i-1,j-k}\times \binom{a_i}{k}\times\frac{1}{k!}
$$
最后 $\sum f_{n,j}(m - j)!$ 即为答案。

## 算法 2

我们观察到上式很像分治 NTT 的式子，我们考虑转化一下。

令 $g_{i,k} = \binom{a_i}{k}\times\frac{1}{k!}$，上边的东西显然可以改写成：
$$
f_{i,j+k} = \sum\limits_{} f_{i-1,j}\times g_{i-1,k}
$$
这个东西显然就是分治 NTT 的东西了，于是做一下时间复杂度 $\mathcal{O}(n\log^2 n)$

```cpp
const ll mod = 998244353;
const ll G = 3;
const ll Gi = 332748118;
ll n,m,a[N];
ll jc[N],inv[N],lim,len,rev[N];

ll ffpow(ll a,ll b) {
    ll ans = 1;
    for (;b;b >>= 1) {
        if (b & 1) ans = ans * a % mod;
        a = a * a  % mod;
    }
    return ans;
}

ll C(int n,int m) {
    if (n > m || n < 0) return 0;
    return jc[m] * inv[n] % mod * inv[m - n] % mod;
}

void NTT(ll *A,int typ) {
    for (int i = 0;i < lim;++i) if (i < rev[i]) swap(A[i],A[rev[i]]);
    for (int mid = 1;mid < lim;mid <<= 1) {
        ll gn = ffpow(typ==1?G:Gi,(mod - 1) / (mid << 1));
        for (int j = 0;j < lim;j += (mid << 1)) {
            ll w = 1;
            for (int k = 0;k < mid;++k,w = (w * gn) % mod) {
                ll x = A[j + k],y = w * A[j + k + mid] % mod;
                A[j + k] = (x + y) % mod;
                A[j + k + mid] = (x - y + mod) % mod;
            }
        }
    }
}

vector <ll> tmp[N];
ll f[N],g[N];
vector <ll> CDQNTT(int l,int r) {
    if (l == r) {
        return tmp[l];
    }
    int mid = (l + r) >> 1;
    vector <ll> L = CDQNTT(l,mid),R= CDQNTT(mid + 1,r);
    int l1 = L.size(),l2 = R.size();
    lim = 1,len = 0;
    while (lim < l1+l2) lim <<= 1,++len;
    for (int i = 0;i < lim;++i) rev[i] = (rev[i>>1] >> 1) | ((i & 1) << (len - 1));
    for (int i = 0;i < l1;++i) f[i] = L[i];
    for (int i = l1;i < lim;++i) f[i] = 0;
    for (int i = 0;i < l2;++i) g[i] = R[i];
    for (int i = l2;i < lim;++i) g[i] = 0;
    NTT(f,1);NTT(g,1);
    for (int i = 0;i < lim;++i) f[i] = f[i] * g[i] % mod;
    NTT(f,-1);
    ll inv = ffpow(lim,mod - 2);
    for (int i = 0;i <= lim;++i) f[i] = f[i] * inv % mod;
    L.clear();
    for (int i = 0;i <= l1 + l2 -2;++i) L.push_back(f[i]);//,printf("%lld ",f[i]);
    return L;
}

signed main() {
    read(n,m);
    jc[0] = 1;
    for (int i = 1;i <= n;++i) read(a[i]);
    for (int i = 1;i <= m;++i) jc[i] = jc[i-1] * i % mod;
    inv[m] = ffpow(jc[m],mod-2);
    for (int i = m-1;i >= 0;--i) inv[i] = (inv[i+1] * (i + 1)) %mod;
    for (int i = 1;i <= n;++i) {
        for (int k = 0;k <= a[i];++k) {
            tmp[i].push_back(C(k,a[i])* inv[a[i]-k] % mod);
        }
    }
    vector <ll> res = CDQNTT(1,n);
    ll ans = 0;
    for (int i = 0;i <= m;++i) {
        if (i & 1) ans = (ans + res[i] * (mod - 1) % mod * jc[m - i] % mod) % mod;
        else ans = (ans + res[i] * jc[m-i] % mod) % mod;
    }
    printf("%lld\n",ans % mod);
	return 0;
}

```

# C

口胡一下：