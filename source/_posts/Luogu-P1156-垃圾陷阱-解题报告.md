title: Luogu P1156 垃圾陷阱 解题报告
categories: 解题报告
tags: [DP,背包,线性DP]
date: 2020-07-02 13:57:42
---

> [P1156 垃圾陷阱](https://www.luogu.com.cn/problem/P1156)
>
> “垃圾井”是农夫们扔垃圾的地方，它的深度为$D(2 \le D \le 100)$英尺。
>
> 卡门想把垃圾堆起来，等到堆得与井同样高时，她就能逃出井外了。另外，卡门可以通过吃一些垃圾来维持自己的生命。
>
> 每个垃圾都可以用来吃或堆放，并且堆放垃圾不用花费卡门的时间。
>
> 假设卡门预先知道了每个垃圾扔下的时间$t (0 < t \le 1000)$，以及每个垃圾堆放的高度$h(1 \le h \le 25$)和吃进该垃圾能维持生命的时间$f(1 \le f \le 30)$，要求出卡门最早能逃出井外的时间，假设卡门当前体内有足够持续$10$小时的能量，如果卡门$10$小时内没有进食，卡门就将饿死。


<!--more-->
### 思路：

线性 DP + 背包变形

回顾一下 01 背包的基本思路：

对于每个物品，有两种决策方式

1. 选择该物品 $f_{i,j} = \max(f_{i,j},f_{i,j-v_i}+w_i)$
2. 不选择该物品 $f_{i,j} = \max(f_{i,j},f_{i-1,j})$

对应本题，每个物品同样有两种决策方式

1. 吃下该物品，则血量增加 $w_i$
2. 堆放该物品，血量减 1，高度增加 $h_i$

所以我们可以设计 $f_{i,j}$ 表示当前对第 $i$ 个物品进行决策，已经填满 $j$ 的高度。

物品还带有时间维，所以我们可以预先对所有物品进行排序，保证物品这维增长是线性的。

转移比较难思考，第一种转移是从 $f_{i-1,j}$ 来的，而第二种转移是要到 $f_{i,j+h_i}$ 去，所以写的方式还有些不同。

转移方程：

$$len = time_i - time_i-1$$

表示从上一个物品到当前物品的时间

$$f_{i,j} = \max(f_{i,j}.f_{i-1,j} -len + w_i)$$

表示吃下该物品的转移，注意中间的时间也要扣掉

$$f_{i,j+h_i} = \max(f_{i,j+h_i,f_{i-1,j} - len})$$

表示堆放该物品的转移。

初值： $f_{0,0} = 10$


在这里，我们并不需要倒序循环 $j$ （因为第一维循环保证用$i-1$阶段更新$i$），但是如果需要用滚动数组优化的话，为了避免重复更新需要倒序循环。

并且在转移过程中，如果当前血量足够**下一次**跳就可以跳出陷阱，就可以直接终止循环，记录答案了。因为从小到大枚举 $i$ 所以可以保证答案最优。

最后，如果实在填不满的话，从头开始模拟一遍即可。

### 代码：

```cpp
#include <algorithm>
#include <cstring>
#include <cstdio>
#include <vector>
#include <cctype>
#include <cmath>
#include <unordered_map>
#define ll long long
#define ri register int

char buf[1 << 20], *p1, *p2;
//#define getchar() (p1 == p2 && (p2 = (p1 = buf) + fread(buf, 1, 1 << 20, stdin), p1 == p2)?EOF: *p1++)
template <typename T> inline void read(T &t) {
	ri v = getchar();T f = 1;t = 0;
	while (!isdigit(v)) {if (v == '-')f = -1;v = getchar();}
	while (isdigit(v)) {t = t * 10 + v - 48;v = getchar();}
	t *= f;
}
template <typename T,typename... Args> inline void read(T &t,Args&... args) {
	read(t);read(args...);
}

template <typename T> inline T min(T x,T y) {return x<y?x:y;}
template <typename T> inline T max(T x,T y) {return x>y?x:y;}
const int INF = 0x3f3f3f3f;
const ll inf = 0x3f3f3f3f3f3f3f3fLL;
const int N = 2010;
int f[N][N],n,m;
struct node {
	int t,f,h;
	friend inline bool operator < (const node &a,const node &b) {
		return a.t < b.t;
	}
}a[N];
bool flag;
int main() {
	read(m,n);
	for (int i = 1;i <= n;++i) {
		read(a[i].t,a[i].f,a[i].h);
	}
	std::sort(a+1,a+1+n);
	memset(f,0xcf,sizeof f);
	f[0][0] = 10;
	for (int i = 1;i <= n;++i) {
		if (flag) break;
		for (int j = 0;j <= m;++j) {	
			if (f[i-1][j] - (a[i].t - a[i-1].t) >= 0) {
				if (j + a[i].h >= m) {
					printf("%d",a[i].t);
					flag = 1;break;
				}
				f[i][j] = max(f[i][j],f[i-1][j] - (a[i].t - a[i-1].t) + a[i].f);
				f[i][j+a[i].h] = max(f[i][j+a[i].h],f[i-1][j] - (a[i].t - a[i-1].t));
			}
			
		}
	}
	int ans = 0,now = 10;
	if (!flag) {
		for (int i = 1;i <= n;++i) {
			if (now < (a[i].t - a[i-1].t)) {
				printf("%d",now+ans);
				return 0;
			}
			ans += (a[i].t - a[i-1].t);
			now = now - (a[i].t - a[i-1].t) + a[i].f;
		}
		printf("%d",now +ans);
	}
	
	return 0;
} 
```

