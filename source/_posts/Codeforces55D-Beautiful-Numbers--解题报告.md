---
title: Codeforces55D-Beautiful Numbers  解题报告
categories: 解题报告
tags: [数论,DP,数位DP]
date: 2019-08-13 11:09:00
---
> [Codeforces55D-Beautiful Numbers](http://codeforces.com/problemset/problem/55/D)

> 定义“美丽的数字”为：被它自己的每一位数上的数整除的数

> 给定区间$[L,R]$，求有多少个美丽的数字

<!--more-->
## 思路

显然的数位DP，简单拉个板子就写完了.但是定义状态的时候发现会存不下，我们考虑压缩一下他们的状态。

引入两条重要的性质：

1. 如果一个数能整除它的所有数位上的数，那么它一定能整除所有这些数位上的数的最小公倍数
2. 而$1$到$9$的最小公倍数是$2520$，所有数可能形成的数位上所有数的最小公倍数一定是$2520$的因数

因为$lcm(1,2,3,4,5,6,7,8,9) = 2520$，所以直接定义$dp[i][j][k]$为前$i$位数$mod$$  2520 = j$，前$i$位数的$lcm$为$j$。

然而，这样设计状态出来会发现总状态数达到了$20*2520*2520 = 127008000$，显然存不下，我们再次考虑进行优化。观察到第二维还是会有冗余，手打个表可以发现总共只有48种可能性，再次进行压缩。

根据性质1，如果最后的数能整除$k$，那么它一定能整除它各位数字的最小公倍数。由于性质2，所有可能出现的$k$都是$2520$的因数，借助这一性质，将$j$取模$2520$，若所得的结果整除$k$，那么原来的数也一定整除$k$。所以转移方程就很自然的出来了：

$f[i][j][k] = \sum f[i][lcm(k,x)][j*10+x \mod 2520|1 \leq x\leq Maxx]$

## 代码

```cpp
#include <cstdio>
#include <cctype>
#include <cstring>
#define ll long long
#define FILE A
#define N 2520 + 1
#define M 200
inline ll read() {
	char v = getchar();ll x = 0,f = 1;
	while (!isdigit(v)) {if (v == '-'){f = -1;}v = getchar();}
	while (isdigit(v)) {x = x * 10 + v - 48;v = getchar();}
	return x * f;
}
ll dp[20][50][N],dig[N],ind[N],cnt,mod,num[N],l,r,c;ll gcd(int a,int b){return b?gcd(b,a%b):a;}
ll mylcm(int a,int b){return a/gcd(a,b)*b;}
ll dfs(ll now,ll lcm,ll sum,int limit) {
	if (!now) {
		return sum % lcm == 0;
	}
	if (!limit && dp[now][num[lcm]][sum] != -1) return dp[now][num[lcm]][sum];
	ll ans = 0,ret = (limit)?dig[now]:9;
	for (ll i = 0;i <= ret;++i) {
		int lcm1 = i ? mylcm(lcm,i) : lcm;
		int sum1 = (sum * 10 + i) % mod;
		ans += dfs(now-1,lcm1,sum1,limit&&i==ret);
	} 
	if (!limit) {
		dp[now][num[lcm]][sum] = ans;
	}
	return ans;
}
ll work(ll x) {
	int len = 0;
	while (x) {
		dig[++len] = x % 10;x /= 10;
	}
	return dfs(len,1,0,1);
}
int main() {
	memset(dp,-1,sizeof(dp));
	int T = read();
	mod = 2520;
	for (int i = 1;i <= mod;++i) {
		if (mod % i == 0) {
			num[i] = cnt++;
		} 
	}
	while (T--) {
		ll l = read(),r = read();
		printf("%lld\n",work(r) - work(l-1));
	}
	return 0;
}
```

