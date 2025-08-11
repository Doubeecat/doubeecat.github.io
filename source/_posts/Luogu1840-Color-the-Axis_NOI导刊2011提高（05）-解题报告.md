title: Luogu1840 Color the Axis_NOI导刊2011提高（05） 解题报告
categories: 解题报告
tags: [线段树]
date: 2019-09-22 03:12:00
---
> [Color the Axis_NOI导刊2011提高（05）](<https://www.luogu.org/problem/P1840>)

> 在一条数轴上有$N$个点，分别是$1 \rightarrow N$。一开始所有的点都被染成黑色。接着我们进行$M$次操作，第$i$次操作将$[L_i,R_i]$这些点染成白色。请输出每个操作执行后剩余黑色点的个数。

<!--more-->
## 解题思路：

这个题的思路很简单，把黑色的部分当成1，白色的部分当成0来处理，使用支持区间修改，区间查询的数据结构维护一下即可。  

如果你和我一样是写线段树的，那么更简单了，在下传lazytag的时候只需要将其的左右节点清零即可。

另外在区间查询时由于每次都是查整段的，所以只需要输出`tree[1].num`即可

## 代码：

```cpp
#include <cstdio>
#include <cctype>
#include <algorithm>
using namespace std;
const int N = 200010;
const int M = 500010<<1;
inline int read() {
	int x = 0,f = 1;char v = getchar();
	while (!isdigit(v)) {if (v =='-') f = -1;v = getchar();}
	while (isdigit(v)) {x = x * 10 + v - 48;v = getchar();}
	return x * f;
}

struct node {
	int l,r,num,tag;
}tree[N<<2];

inline void build(int p,int l,int r) {
	tree[p].l = l;tree[p].r = r;
	if (l == r) {
		tree[p].num = 1;
		return ;
	}
	int mid = (l + r) >> 1;
	build (p << 1,l,mid);
	build (p << 1 | 1,mid + 1,r);
	tree[p].num = tree[p<<1].num + tree[p << 1 | 1].num;
	return;
}	

inline void spread(int p) {
	if (!tree[p].num) {
		tree[p<<1].num = tree[p<<1|1].num = 0;
	}
}

inline void change (int p,int x,int y) {
	if (tree[p].l >= x && tree[p].r <= y) {
		tree[p].num = 0;
		return ; 
	}
	spread(p);
	int mid = (tree[p].l + tree[p].r) >> 1;
	if (mid >= x) change(p<<1,x,y);
	if (mid < y) change(p<<1|1,x,y);
	tree[p].num = tree[p<<1].num + tree[p<<1|1].num;
	return ;
}

int main() {
	int n = read(),m = read();
	build (1,1,n);
	for (int i = 1;i <= m;++i) {
		int x = read(),y = read();
		change(1,x,y);
		printf("%d\n",tree[1].num);
	}
	return 0;
}
```
