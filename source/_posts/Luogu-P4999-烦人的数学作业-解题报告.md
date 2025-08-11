title: Luogu P4999 烦人的数学作业 解题报告
categories: 解题报告
tags: [DP,数位DP]
date: 2019-09-07 15:48:00
---
> [P4999 烦人的数学作业](<https://www.luogu.org/problem/P4999>)

> 给出一个区间$L - R$，求$L$到$R$区间内**每个数的数字和**，如123这个数的数字和为1+2+3=6

> 有T组数据，结果$\mod 10^9+7$

$（1 \leq L \leq R \leq 10^18）$
<!--more-->
## 解题思路：

~~确实烦人~~

第一眼看上去跟[[ZJOI2010]数字计数](<https://www.luogu.org/problem/P2602>)很像，确实是，把那个题的数位DP抄过来乘个 i 这题就没了。

那为啥要写这个题解呢？因为这个题坑多。

1. 注意开 long long
2. 取模很坑，请使用传统技巧`(ans % mod + mod) % mod`，正确性证明应该不用多说

## 代码：


```cpp
#include <cstdio>
#include <cctype>
#include <cmath>
#include <cstring>
#define int long long 

const int N = 20;
const int mod = 1e9+7;
const int M = N << 1;

inline int read() {
	char v = getchar();int x = 0,f = 1;
	while (!isdigit(v)) {if (v == '-')f = -1;v = getchar();}
	while (isdigit(v)) {x = x * 10 + v - 48;v = getchar();}
	return x * f;
}

int num[N],dp[N][N],l,r;

int dfs(int pos,bool limit,bool zer,int dig,int sum) {
	int ans = 0;
	if (pos == 0) {
		return sum;
	}	
	if (!limit && dp[pos][sum]) return dp[pos][sum];
	int up = 9;
	if (limit) up = num[pos];
	for (int j = 0;j <= up;++j) {
		ans += dfs(pos-1,(j==up)&&limit,zer||j,dig,sum+((j||zer)&&(j==dig)));
	}
	if (!limit&&zer){
		dp[pos][sum] = ans;
	}
	return ans;
}

int work(int p,int w) {
	memset(dp,0,sizeof(dp));
	int len = 0;
	while (p) {
		num[++len] = p % 10;
		p /= 10;
	}
	return dfs(len,1,0,w,0) % mod;
} 

signed main() {
	int T = read();
	while (T--) {
		int ans = 0;
		l = read(),r = read();
		for (int i = 0;i <= 9;++i) {
			ans += ((work(r,i) - work(l-1,i) % mod) * i) % mod;
		}
		printf("%lld\n",(ans%mod+mod)%mod);
	}
	return 0;
}
```

