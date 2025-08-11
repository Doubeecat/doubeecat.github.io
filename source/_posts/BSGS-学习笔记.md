title: BSGS 学习笔记
categories: 学习笔记
tags: [数论,BSGS]
date: 2021-04-05 05:35:21
---

BSGS ~~(北上广深)~~ 算法，用来解决一类 $a ^ x\equiv b \pmod p$ (即离散对数) 的问题。

<!--more-->

---

# 1. 基础 BSGS

我们考虑 $a$ 与 $p$ 互质的情况。既然 $a$ 与 $p$ 互质，我们不难想到欧拉定理 $a^{\varphi{}(p)} \equiv 1\pmod p$，所以 $x$ 的 取值范围为 $[1,\varphi{}(p)]$，直接枚举时间复杂度 $O(p)$

BSGS 算法的灵魂在于，对整个枚举的过程进行了优化。

我们考虑把原式变成这样的形式 $a^{Ax - B} \equiv b\pmod p$，由此可得 $B$ 的取值范围为 $[0,x)$。这里我们显然可以把 $a^B$ 除过去，得到我们想要的目标式子：

$$
    a^{Ax} \equiv b \times a ^ B\pmod p
$$

注意到了 $a,b$ 均是定值，所以我们可以考虑固定三个变量中的两个，然后枚举。所以我们接下来考虑 $A,B$ 之间取值范围的关系。

我们有 $Ax-B\in [1,\varphi{}(p)]$，目标是让 $A+B$ 最小化。根据基本不等式知识易得当 $x = \lceil \sqrt{\varphi{}(p)} \rceil$ 时 $A+B$ 最小。当然我们这里为了避免计算 $\varphi$，所以我们可以取 $A = \lceil \sqrt{p} \rceil$,显然这个 $A$ 取的范围是覆盖原本的计算范围的。

接下来就是整个算法最精华的部分了，我们显然要对左右两边进行枚举，然后找有没有相等的值。我们先对左边的部分进行枚举，并存进一个 hash 表内，每个值对应的是 $A$ 的大小(非 OI 赛场上完全可用 `unordered_map` 代替 hash 表)

```cpp
unordered_map <ll,ll> hash;
ll tmp = b * a % p,t = sqrt(p) + 1;
for (ll i = 1;i <= t;++i) {
    hash[tmp] = i; 
    //printf("%lld %lld\n",tmp,i);
    tmp = (tmp * a) % p;
}
```

然后，我们类似的对右边进行枚举，并且在 hash 表内查询，最后算出答案。

```cpp
ll at = ffpow(a,t,p),now = at;
for (ll i = 1;i <= t;++i) {
    if (hash[now]) {
        return i * t - hash[now];
    }
    now = (now * at) % p;
}
```

时间复杂度 $O(\sqrt{p} \log p)$

Code:

```cpp
ll ffpow(ll a,ll b,ll p) {
	ll ans = 1;
	for (;b;b >>= 1) {
		if (b & 1) ans = ans * a % p;
		a = a * a % p;
	}
	return ans % p;
}

ll BSGS(ll a,ll b,ll p) {
	unordered_map <ll,ll> hash;
	ll tmp = b * a % p,t = sqrt(p) + 1;
	for (ll i = 1;i <= t;++i) {
		hash[tmp] = i; 
		//printf("%lld %lld\n",tmp,i);
		tmp = (tmp * a) % p;
	}
	ll at = ffpow(a,t,p),now = at;
	for (ll i = 1;i <= t;++i) {
		if (hash[now]) {
			return i * t - hash[now];
		}
		now = (now * at) % p;
	}
	return -1;
}
```

# 2. 拓展 BSGS

接下来我们考虑 $a$ 和 $p$ 不互质的情况。首先我们需要先确定是否有解的问题，我们可以把原本的同余方程转化成 
$$
    a ^ x + p\times y = b \\
    a \times a ^{x-1} + p \times y = b
$$

根据贝祖定理，方程有解当且仅当 $b | gcd(a,p)$。

很自然想到把不互质的情况转化成互质的情况。我们设 $d = gcd(a,p)$，将上面方程中的系数不断除以 $d$，假设我们这个过程进行了 $cnt$ 次，我们的式子化成这样

$$
    \frac{a}{d^{cnt}} \times a ^{x-cnt-1} + \frac{p}{d^{cnt}} \times y = \frac{b}{d^{cnt}}
$$

再将其转回我们熟悉的同余方程形式就是

$$
    a ^{x-cnt-1} \equiv \frac{b}{d^{cnt}}\pmod{ \frac{p}{d^{cnt}}}
$$

这样我们的 $a$ 显然与 $\frac{p}{d^{cnt}}$ 互质。

所以对之后的式子再变化一下：

$$
    a ^{Ax} \equiv \frac{b}{d^{cnt}} \times a ^ {b} \times a ^{cnt+1} \pmod {\frac{p}{d^{cnt}}}
$$


套用一次 BSGS 算法即可，本质上是一样的。

Code:

```cpp
ll exBSGS(ll a,ll b,ll p) {
    a %= p,b %= p;
    if (p == 1) return 0;//注意你当 p = 1 时是有解的
    if (b == 1) return 0;
    ll g,cnt = 0,c = 1;
    while ((g = gcd(a,p)) > 1) {
        if (b % g) return -1;
        b /= g,p /= g,c = c * (a / g) % p,++cnt;
        //这里的c实际上是\frac{a}{g} ^ cnt，之后算右半边的时候回用到
        if (b == c) return cnt;
    }
    unordered_map <ll,ll> hash;
	ll tmp = b * a % p,t = sqrt(p) + 1;
	for (ll i = 1;i <= t;++i) {
		hash[tmp] = i; 
		//printf("%lld %lld\n",tmp,i);
		tmp = (tmp * a) % p;
	}
	ll at = ffpow(a,t,p),now = c * at % p;
	for (ll i = 1;i <= t ;++i) {
		if (hash.count(now)) {
			return i * t - hash[now] + cnt;//注意加上次数
		}
		now = (now * at) % p;
	}
	return -1;
}
```

# 3.例题

相信大家一定会马上去把模板题秒了，所以这里就不放了。

> [P3306 [SDOI2013] 随机数生成器](https://www.luogu.com.cn/problem/P3306)
> 
> 给定序列 $x_{i+1} \equiv a \times x_i + b\pmod p$，求最小的 $i$ 使得 $x_i = t$ 成立
> 
> $0 \leq a, b, x_1, t \lt p$，$2 \leq p \leq 10^9$
> 
> $p$ 为质数。


首先我们可以先把前几项推一下：

$$
    x_2 = a \times x_1 + b \\
    x_3 = a^2 \times x_1 + a \times b + b \\
    x_4 = a^3 \times x_1 + a ^ 2 \times b + a \times b + b
$$

所以我们运用一点等比数列的知识，可以推出通项公式。

$$
    x_n = a ^{n-1} \times x_1 + b \times \frac{1-a^{n-1}}{1-a}
$$

接下来就是激情推式子！

$$
    \begin{aligned}
        x_n = a ^{n-1} \times x_1 + b \times \frac{a^{n-1}-1}{a-1} &\equiv t \pmod p \\
        a ^ {n-1} \times x_1 + \frac{b}{a-1}  \times a ^ {n-1} - \frac{b}{a-1} &\equiv t \pmod p \\
        \frac{ax_1+b-x_1}{a-1} \times a ^ {n-1}&\equiv t + \frac{b}{a-1}  = \frac{at - t + b}{a-1}\pmod p \\
        a^{n-1} &\equiv \frac{at-t+b}{ax_1+b-x_1}\pmod p\\
        a^{n-1} &\equiv \frac{t-at-b}{x_1-ax_1-b}\pmod p
    \end{aligned}
$$

注意到 $p$ 是质数，所以可以直接用普通 BSGS 求即可。

代码实现上细节比较多，自行查看。

Code:

```cpp
void solve() {
    ll p,a,b,x1,t;read(p,a,b,x1,t);
    if (x1 == t) {puts("1");return ;}
    if (a == 0) {
        if (b == t) {puts("2");}
        else {puts("-1");}
        return;
    }
    if (a == 1) {
        if ((t - x1) % gcd(b,p)) {
            puts("-1");
            return ;
        }
        t = (t - x1 + p) % p;
        printf("%lld\n",(t * ffpow(b,p-2,p)) % p + 1);
        return ;
    }
    ll r = ((t - a * t - b) % p * ffpow((x1 - x1 * a - b) % p,p-2,p) % p) % p;
    ll ans = BSGS(a,r,p);
    if (ans == -1) puts("-1");
    else printf("%lld\n",ans + 1);
}
```
