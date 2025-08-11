---
title: Luogu 1417 烹调方案 解题报告
categories: 解题报告
tags: [DP,背包]
date: 2019-09-22 03:13:00
---
> [烹调方案](<https://www.luogu.org/problem/P1417>)

> 一共有$n$件食材，每件食材有三个属性， $a_i$ ， $b_i$ 和 $c_i$ ，如果在 $t$ 时刻完成第 $i$ 样食材则得到 $a_i-t*b_i$ 的美味指数，用第 $i$ 件食材做饭要花去 $c_i$ 的时间。

> 在 $T$ 时间内设计烹调方案使得美味指数最大

<!--more-->
## 解题思路：

显然是一个01背包，但是，当你写完01背包板子交上去以后会发现只有30分。  

题目中给了一个条件：在 $t$ 时刻完成第 $i$ 样食材则得到 $a_i-t*b_i$ 的美味指数，有了这个条件，显然我们要对每个物品选择的先后性进行讨论了。

设 $a_1,b_1,c_1$ 为第一组物品，$a_2,b_2,c_2$ 为第二组物品， $t$为当前时间.

将两个物品可获得的值表示出来为：

$$
m_1 = a_1-(t \times c_1) \times b_1 + a_2 - (t+c_1+c_2) \times b_2
$$

$$
m_2 = a_2-(t \times c_2) \times b_2 + a_1 - (t+c_1+c_2) \times b_1
$$

设 $m_1 > m_2$ 得

$$
a_1-(t \times c_1) \times b_1 + a_2 - (t+c_1+c_2) \times b_2   > m_2 = a_2-(t \times c_2) \times b_2 + a_1 - (t+c_1+c_2) \times b_1
$$

拆项得

$$
a_1-b_1t-b_1c_1+a_2-b_2t-b_2c_1-b_2c_2>a_2-b_2t-b_2c_2+a_1-b_1t-b_1c_1-b_1c_2
$$

化简得  

$$
b_1c_2 > b_2c_1
$$


由此，我们就得到了我们 sort 函数中cmp的写法。注意开 long long 这题就完了。

这题的关键就在此，我不知道是为什么，大部分题解中都没有给出自己的完整证明，这个证明也不难证，希望读者看完后可以自己证明一遍。
## 代码：

```cpp
#include <cstdio>
#include <cctype>
#include <algorithm>
#define int long long
using namespace std;
const int N = 1000100;
const int M = 500010<<1;
inline int read() {
	int x = 0,f = 1;char v = getchar();
	while (!isdigit(v)) {if (v =='-') f = -1;v = getchar();}
	while (isdigit(v)) {x = x * 10 + v - 48;v = getchar();}
	return x * f;
}
int f[N],n,m,ans;
struct node {
	int a,b,c;
	friend inline bool operator < (const node &rhs,const node &rs) {
		return rhs.b*rs.c > rhs.c*rs.b;	
	}
}pre[N];
signed main() {
	m = read(),n = read();
	for (int i = 1;i <= n;++i)	pre[i].a = read();
	for (int i = 1;i <= n;++i)	pre[i].b = read();
	for (int i = 1;i <= n;++i)	pre[i].c = read();
	sort(pre+1,pre+1+n);
	for (int i = 1;i <= n;++i) {
		for (int j = m;j >= pre[i].c;--j) {
			f[j] = max(f[j],f[j-pre[i].c] + (pre[i].a - j * pre[i].b));
		}
	}
	for (int i = 1;i <= m;++i) {
		ans = max(ans,f[i]);
	}
	printf("%lld",ans);
	return 0;
}
```
