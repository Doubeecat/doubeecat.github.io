---
title: CF1416C XOR Inverse 解题报告
categories: 解题报告
tags: []
date: 2021-07-05 02:15:00
---
> [CF1416C XOR Inverse](http://codeforces.com/problemset/problem/1416/C)
>
> 给定长度为 $n$ 的数列 $\{a_n\}$，请求出最小的整数 $x$ 使 $\{a_n\oplus x\}$ 的逆序对数最少
> 
> $n\le3\times 10^5,0\le a_n\le 10^9$


<!--more-->

## 解题思路：

~~和学长一起打的训练赛，喜提 rank 27.~~

首先观察到跟异或有关，就立刻要想到 **0-1 trie**。

把所有数丢到 0-1 trie 上开始考虑怎么处理。

我们思考一下怎么才会产生逆序对，在我们顺序插入的过程中，有一个性质是**右子树上的数是严格大于左子树上的数**。

这启发我们在顺序插入的过程中记录一下经过每个点的数个数，记为 $siz_p$，那么接下来考虑按层进行一个贪心构造。

$f_{i,0/1}$ 表示在第 $i$ 层走 $0$ 或者 $1$ 时产生的逆序对个数。

假设我们当前插入数的二进制下第 $i$ 位为 $tmp$，则逆序对数会增加 $siz_{trie_{p}{tmp \oplus 1}}$，并且这个逆序对是因为你走了 $tmp \oplus 1$ 产生的，所以我们就有

$f_{i,tmp \oplus 1} += siz_{trie_{p}{tmp \oplus 1}}$

接下来我们按层处理，非常简单，只要比较一下当前层 $f_{i,0},f_{i,1}$ 哪个更小走哪边即可，注意如果走了 $0$，实际上是表示我们 $x$ 当前这位为 $1$，所以需要加上 $2^{dep}$.

考场上比较傻逼加了一个归并排序，实际上不用，走哪边加上哪边的 $f$ 就可以了。

时间复杂度 $O(n \log n)$

## 代码：

```cpp


const int N = 3e5 + 10;
const int K = 31;

int trie[N*K][2],tot,f[N*K][2],siz[N*K*2];

void insert(int x) {
    int p = 0;
    for (int i = 30;i >= 0;--i) {
        int tmp = ((x >> i)& 1);
        if (!trie[p][tmp]) trie[p][tmp] = ++tot;
        f[i][tmp ^ 1] += siz[trie[p][tmp ^ 1]];
        p = trie[p][tmp];
        ++siz[p];
    }
    return ;
}

int n,a[N],b[N],x,ans,tmp[N];

void merge(int l, int mid, int r) {
	if (l == r) return;
	if (l + 1 == r) {
		if (b[l] > b[r]) {
			ans++;
			swap(b[l], b[r]);
		}
		return;
	}
	merge(l, (l + mid) >> 1, mid);
	merge(mid + 1, (mid + 1 + r) >> 1, r);
	int i = l, j = mid + 1;
	for (int k = l; k <= r; k++)
		if (j > r || (i <= mid && b[i] <= b[j])) tmp[k] = b[i++];
		else {
			tmp[k] = b[j++];
			ans += mid - i + 1;
		}
	for (int k = l; k <= r; k++) b[k] = tmp[k];
}

signed main() {
    read(n);
    for (int i = 1;i <= n;++i) read(a[i]),insert(a[i]);
    for (int i = 30;i >= 0;--i) {
        if (f[i][1] > f[i][0]) {
            x += (1 << i);
        }
    }
    for (int i = 1;i <= n;++i) b[i] = a[i] ^ x;
    merge(1,(1 + n) >> 1,n);
    printf("%lld %lld\n",ans,x);
    return 0;
}
```