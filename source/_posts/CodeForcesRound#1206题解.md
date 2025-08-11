---
title: CodeForcesRound#1206题解
categories: Codeforces赛后题解
tags: [图论,暴力,codeforces,思维]
date: 2019-08-22 13:32:00
---
##A. [Choose Two Numbers](<https://codeforces.com/contest/1206/problem/A>)
### 题目描述：

给你两个集合 $A,B$, 要求你分别从$A,B$中取出一个数，使得两数之和不在$A$中也不在$B$中

$n ,m\leq 100$
<!--more-->
### 解题思路：

看数据范围应该很容易想出来一个 $O(nm(n+m))$ 的暴力，枚举 $A,B$ 中的每一个数，将他们相加之后再在两个集合中判定一下，这个题就完了。事实上由于$n ,m\leq 100$ 可以轻松通过本题。

但是显然，还会有更好的解法，我们可以观察到，如果选取这两个集合中最大的数相加，显然这个和不会出现在两个集合里面，复杂度下降到了 $O(m \log {m} + n \log {n})  $  如果用排序还可以达到$O(m+n)$的线性时间复杂度(排序是作者给出的解法，实际上可以直接读入的时候求出)

### 代码：

 $O(nm(n+m))$

```cpp
#include <cstdio>
#include <cctype>

inline int read() {
	char v = getchar();int x = 0,f = 1;
	while (!isdigit(v)) {if(v == '-')f = -1;v = getchar();}
	while (isdigit(v)) {x = x * 10 + v - 48;v = getchar();}
	return x * f;
}

const int N = 110;

int a[N],b[N],n,m,a1,a2;

int main() {
	n = read();
	for (int i = 1;i <= n;++i) {
		a[i] = read();
	}
	m = read();
	for (int i = 1;i <= m;++i) {
		b[i] = read();
	}
	for (int i = 1;i <= n;++i) {
		for (int j = 1;j <= m;++j) {
			a1 = a[i],a2 = b[j];
			int sum = a1 + a2;bool flag = 0;
			for (int k = 1;k <= n;++k) {
				if (sum == a[k]) {
					flag = 1;
					break;
				}
			}
			for (int k = 1;k <= m;++k) {
				if (sum == b[k]) {
					flag = 1;
					break;
				}
			}
			if (!flag) {
				printf("%d %d\n",a1,a2);
				return 0;
			}
		}
	}
	return 0;
}
```

 $O(m + n)  $ 

```cpp
#include <cstdio>
#include <cctype>
#include <algorithm>
using std::max;
inline int read() {
	char v = getchar();int x = 0,f = 1;
	while (!isdigit(v)) {if(v == '-')f = -1;v = getchar();}
	while (isdigit(v)) {x = x * 10 + v - 48;v = getchar();}
	return x * f;
}

const int N = 110;
int tmp,a,b;

int main() {
	n = read();
	for (int i = 1;i <= n;++i) {
		tmp = read();
        a = max(a,tmp);
	}
	m = read();
	for (int i = 1;i <= m;++i) {
		tmp = read();
        b = max(b,tmp);
	}
    printf("%d %d",a,b);
	return 0;
}
```

##B. [Make Product Equal One](https://codeforces.com/contest/1206/problem/B)

### 题目描述：

给你一个有 $n$ 个数字的数列，你有两种操作：将某个数 $+1$ 或 $-1$ 。求用最小的操作次数让这个数列的每一项乘起来等于 1

$n \leq 10^5$

### 解题思路：

~~用C换来的~~

分析一下，每个数字只有两种情况：变成 1 或者变成 -1 

所以每个数字的代价就是`min(abs(1 - a[i]),abs(-1 - a[i]))`

值得一提的几个~~大坑~~点是，最后可能会变成 -1 ，所以顺便记一下，如果最后变成 -1 了记得加上 2 。而且 0 也要记，最后要加上 0 的个数。

时间复杂度$O(n)$

### 代码：

```cpp
#include <cstdio>
#include <cctype>
#include <cmath>
#include <algorithm>
#include <iostream>
using namespace std;
#define int long long
inline int read() {
	char v = getchar();int x = 0,f = 1;
	while (!isdigit(v)) {if(v == '-')f = -1;v = getchar();}
	while (isdigit(v)) {x = x * 10 + v - 48;v = getchar();}
	return x * f;
}

const int N = 100010;

int pre[N],n,p1,p2,ans,z,res = 1;

signed main() {
	n = read();
	for (int i = 1;i <= n;++i) {
		pre[i] = read();
		if (pre[i] == 0) {
			z++;
		}
	}
	sort(pre+1,pre+1+n);
	for (int i = 1;i <= n;++i) {
		if (pre[i] <= -1) {
			ans += (-1 - pre[i]);
			res *= -1;
		}
		else if (pre[i] >= 1) {
			ans += (pre[i] - 1);
			res *= 1;
		}
	}
	if (res == -1) {
		if (z) {
			ans += z;
		}
		else {
			ans += 2;
		}
	}
	else {
		ans += z;
	}
	cout << ans;
	
	return 0;
}
```

## C. [Almost Equal](<https://codeforces.com/contest/1205/problem/A>)

### 题目描述：

给定一个数字 $n$ ，要求你使用 $1...2*n$ 的数字构造出一个环，使得这个环每三个数相加的和之间差不超过 1.

### 解题思路：

偶数显然不能构造，奇数按照样例构造即可，具体看代码

### 代码：

```cpp
#include <cstdio>
#include <cctype>
#include <cmath>
#include <algorithm>
using std::min;
inline int read() {
	char v = getchar();int x = 0,f = 1;
	while (!isdigit(v)) {if(v == '-')f = -1;v = getchar();}
	while (isdigit(v)) {x = x * 10 + v - 48;v = getchar();}
	return x * f;
}

const int N = 200010;

int pre[N],n,now = 1;

int main() {
	n = read();
	if (n % 2 == 0) {
		printf("NO");
		return 0;
	}
	for (int i = 1;i <= 2*n;++i) {
		pre[now] = i;
		if (i % 2 == 0) {
			now += 1;
		}
		else {
			if ((i / 2) % 2) {
				now -= n;
			}
			else {
				now += n;
			}
		}
	}
	printf("YES\n");
	for (int i = 1;i <= 2*n;++i) {
		printf("%d ",pre[i]);
	}
	return 0;
}
```

## D. [Shortest Circle](<https://codeforces.com/contest/1205/problem/B>)

### 题目描述：

给你 $n$ 个点，每个点有一个点权，两点之间有边当且仅当 $a[i] \& a[j] \ne 0$ ，求出图中最小环的长度。

$n \leq 100000,a_i \leq 10^{18}$

### 解题思路：

直接构造显然是 $O(n^2)$ 的，考虑优化建边的过程。

首先可以观察到，如果二进制内某一位有 3 个 1 ，那么肯定有一个长度为 3 的环，直接输出即可。

那么现在问题简化为了每位至多有两个 1 的情况，我们可以发现每一位最多只有一条边，所以整张图其实至多剩下 60 条边了！接下来直接跑一遍最小环即可，我使用的是 Floyd .

注意开 long long

### 代码：

```cpp
#include <cstdio>
#include <cctype>
#define min(a,b) ((a) < (b) ? (a) : (b))
#define int long long
const int N = 100010;
const int M = 130;
 
inline int read() {
	char v = getchar();int x = 0,f = 1;
	while (!isdigit(v)) {if (v == '-')f = -1;v = getchar();}
	while (isdigit(v)) {x = x * 10 + v - 48;v = getchar();}
	return x * f;
}
 
int a[N],pre[N],f[M][M],n,cnt,dis[M][M],ans = 0x3f3f3f3f;
signed main() {
	n = read();
	for (int i = 1;i <= n;++i) {
		a[i] = read();
		if (a[i] != 0) pre[++cnt] = a[i];
	}
	for (int i = 1;i <= 64;++i) {
		int cnt = 0;
		for (int j = 1;j <= n;++j) {
			if (a[j]>>(i-1)&1) {
				cnt++;
			}
		}
		if (cnt >= 3) {
			printf("3");
			return 0;
		}
	}
	for (int i = 1;i <= cnt;++i) {
		for (int j = 1;j <= cnt;++j) {
			if (i != j && (pre[i] & pre[j]) != 0) {
				f[i][j] = 1;dis[i][j] = 1;
			}
			else {
				f[i][j] = dis[i][j] = 0x3f3f3f3f;
			}
		}
	}
	for (int k = 1;k <= cnt;++k) {
		for (int i = 1;i < k;++i) {
			for (int j = i+1;j < k;++j) {
				ans = min(dis[i][j] + f[i][k] + f[k][j],ans);
			}
		}
		for (int i = 1;i <= cnt;++i) {
			for (int j = 1;j <= cnt;++j) {
				dis[i][j] = min(dis[i][k]+dis[k][j],dis[i][j]);
			}
		}
	}
	printf("%d\n",((ans>=0x3f3f3f3f) ? -1 : ans));
}
```