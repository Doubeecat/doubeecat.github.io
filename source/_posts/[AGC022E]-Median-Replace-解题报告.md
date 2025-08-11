---
title: AGC022E Median Replace 解题报告
categories: 解题报告
tags: [DP,计数问题]
date: 2021-08-11 02:28:00
---
> [AGC022E Median Replace](https://www.luogu.com.cn/problem/AT3950)
> 
> 给出一个长度为奇数 $n$ 的残缺 $01$ 串，问有多少种补全方法，每次将连续三个位替换为它们的中位数后，能有一种方案使它变为 $1$。
>
> $n\leq 3\times 10^5$

<!--more-->

## 解题思路：

考虑如何判断合法，题目操作本质是 $000$ 变为 $0$，$111$ 变为 $1$，删去 $10,01$

维护一个栈，从左往右加入每一个字符，如果栈顶为 $0$ 那么就将 $0$ 留下，直到有连续 3 个 0 的时候删掉两个。

若新加入了 1，删去栈顶的 0 和新加入的 1。如果栈顶为 1 ，那么不管加入什么都留下。通过这个可以得出新的 $j',k'$

栈里必定是若干 $1$ 接上若干 $0$，当 $1$ 个 $1$ 的段数大于等于 $2$ 个 $0$ 的段数或者最后只有 1 的时候，串就是美丽的。

定义 $f_{i,j,k}$ 表示考虑前 $i$ 位，当前有 $j$ 个 $1$ 和 $k$ 个 $0$ 的方案数，枚举❓是 $0/1$ 转移。
那么转移需要分类讨论一下了：

- 若加入的是 $1$
    - 若栈里已经有至少 $1$ 个 $0$，直接抵消，$f_{i-1,j,k} \to f_{i,j,k-1}$ （$01$ 无用）
    - 否则，若栈里有 $2$​​ 个 $1$​​，加入后无影响不改变 $f_{i-1,2,k} \to f_{i,2,k}$​​ 
    - 否则，栈里 $1$​​ 个数 $+1$​​ $f_{i-1,j,k} \to f_{i,j+1,k}$​​ 

- 若加入的是 $0$ 

    - 若栈里已有 $2$ 个 $0$ 消成 $1$ 个，$f_{i-1,j,2} \to f_{i,j,1}$
    - 否则栈里 $0$ 个数 $+1$ $f_{i-1,j,k} \to f_{i,j,k+1}$

- 若加入的是 $?$

    分别当成 $0/1$ 转移即可。

不难发现我们的 $j,k \leq 2$ 代码并不难写。

时间复杂度 $O(n)$


## 代码：

```cpp
char s[N];
int f[N][3][3],n,ans;

signed main() {
    #ifndef ONLINE_JUDGE
        freopen("test.in","r",stdin);
        freopen("test.out","w",stdout);
    #endif
    f[0][0][0] = 1;
    scanf("%s",s+1);
    n = strlen(s+1);
    for (int i = 1;i <= n;++i) {
        for (int j = 0;j <= 2;++j) {
            for (int k = 0;k <= 2;++k) {
                if (s[i] == '0') {
                    if (k < 2) f[i][j][k+1] = (f[i][j][k+1] + f[i-1][j][k]) % mod;
                    else f[i][j][1] = (f[i][j][1] + f[i-1][j][k]) % mod;
                }
                if (s[i] == '1') {
                    if (k) {
                        f[i][j][k-1] = (f[i][j][k-1] + f[i-1][j][k]) % mod;
                    } else {
                        if (j < 2) f[i][j+1][k] = (f[i][j+1][k] + f[i-1][j][k]) % mod;
                        else f[i][2][k] = (f[i][2][k] + f[i-1][j][k]) % mod;
                    }
                }
                if (s[i] == '?') {
                    if (k < 2) f[i][j][k+1] = (f[i][j][k+1] + f[i-1][j][k]) % mod;
                    else f[i][j][1] = (f[i][j][1] + f[i-1][j][k]) % mod;
                    
                    if (k) {
                        f[i][j][k-1] = (f[i][j][k-1] + f[i-1][j][k]) % mod;
                    } else {
                        if (j < 2) f[i][j+1][k] = (f[i][j+1][k] + f[i-1][j][k]) % mod;
                        else f[i][2][k] = (f[i][2][k] + f[i-1][j][k]) % mod;
                    }
                }
            }
        }
    }
    for (int i = 0;i <= 2;++i) {
        for (int j = 0;j <= i;++j) {
            ans = (ans + f[n][i][j]) % mod;
        }
    }

    printf("%lld\n",ans);
    #ifndef ONLINE_JUDGE
        fclose(stdin);fclose(stdout);
    #endif
	return 0;
}

```