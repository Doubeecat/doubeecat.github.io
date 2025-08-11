title: SHCPC2022 I 题解
categories: 解题报告
tags: []
date: 2025-03-23 12:26:26
---
爱讨论！

<!--more-->

## [SHCPC2022 I](https://codeforces.com/group/DndpRd57RB/contest/595309/problem/I)

关注到这个题目本质上等价于下面这个题目：

> 每次随机选择两个点，将其连边，当出现以下情况时就停止连边：
>
> 1. 某个点度数 $>2$
> 2. 重边
> 3. 自环
>
> 求期望步数。

这个问题里，我们要考虑的就是环的情况。我们考虑仅有点数 $\geq 3$ 的链可以成环。所以我们考虑：孤立点，仅有2个点的短链，有大于2个点的长链三种情况。设计状态为 $f_{i,j,k}$ 分别代表孤立点有 $i$ 个，短链 $j$ 条，长链 $k$ 条。接下来我们开始分类讨论：

1. 两个孤立点：

   当 $i>1$ 时，有 $f_{i,j,k}\leftarrow f_{i-1,j+1,k}$，对应概率为 $p = \frac{\binom{n}{2}}{n^2}$

2. 一个孤立点和一条短链：

   考虑拿孤立点和短链的一端成长链，当 $i>0,j>0$ 时，有 $f_{i,j,k}\leftarrow f_{i-1,j-1,k+1}$，对应概率为 $p = \frac{2ij}{n^2}$

3. 一个孤立点和一条长链：

   考虑拿孤立点和长链的一端成长链，当 $i>0,k>0$ 时，有 $f_{i,j,k}\leftarrow f_{i-1,j,k}$，对应概率为 $p = \frac{2ik}{n^2}$

4. 一个短链和另一条短链：

   考虑拿短链一端和另一条短链的一端成长链，当 $j>1$ 时，有 $f_{i,j,k}\leftarrow f_{i,j-2,k+1}$，对应概率为 $p = \frac{\binom{j}{2}*2*2*2}{n^2}$ 注意这里顺序可以颠倒所以多一个 

5. 一个短链和一条长链：

   考虑拿短链两端和长的一端成长链，当 $j>0,k>0$ 时，有 $f_{i,j,k}\leftarrow f_{i,j-1,k}$，对应概率为 $p = \frac{2*2*2jk}{n^2}$

6. 一个长链和另一条长链：

   拿长链一端和另一条长链的一端成长链，当 $k>1$ 时，有 $f_{i,j,k}\leftarrow f_{i,j,k-1}$，对应概率为 $p = \frac{\binom{k}{2}*2*2*2}{n^2}$ 注意这里顺序可以颠倒所以多一个 

7. 长链自己成环

   这个就直接当 $k>0$ 时有 $f_{i,j,k} = f_{i,j,k-1}$，对应概率 $p = \frac{2k}{n}$

转移方程是 $f_{i,j,k} = \frac{1}{\sum p} (f_{sta}\times p)$ 这个是因为考虑非法概率是 $1 - \sum p$，也就是只有 $\sum p$ 的概率继续走下去。根据期望也就是走 $\frac{1}{\sum p}$ 次才可以。代码使用记忆化搜索实现

```cpp
/*
Undo the destiny.
*/
#include <bits/stdc++.h>
using namespace std;
#define ll long long
#define FO(x) {freopen(#x".in","r",stdin);freopen(#x".out","w",stdout);}
#define pii pair<int,int>
#define pll pair<ll,ll>
#define mp make_pair
const int N = 210;
double f[N][N][N];
double n;

double dp(int i,int j,int k) {
    // cerr << i << " " << j << " " << k << ":" << f[i][j][k]<< "\n";
    if (f[i][j][k] != -1) return f[i][j][k];
    if ((i == 1 && j == 0 && k == 0)) return f[i][j][k] = 0;
    if ((i == 0 && j == 1 && k == 0)) return f[i][j][k] = 0;
    if ((i == 0 && j == 0 && k == 0)) return f[i][j][k] = 0;

    double ans = 1;
    double sump = 0,nowp = 0;
    if (i >= 2) {
        nowp = 1.0 * (i-1)*i / (n*n);
        ans += dp(i-2,j+1,k) * nowp;
        sump += nowp;
    }
    if (i >= 1 && j >= 1) {
        nowp = 1.0*i*j*2*2 / (n*n);
        ans += dp(i-1,j-1,k+1) * nowp;
        sump += nowp;
    }
    if (i >= 1 && k >= 1) {
        nowp = 4.0*i*k / (n*n);
        ans += dp(i-1,j,k) * nowp;
        sump += nowp;
    }
    if (j >= 2) {
        nowp = 4.0*j*(j-1) / (n*n);
        ans += dp(i,j-2,k+1) * nowp;
        sump += nowp;
    }
    if (j >= 1 && k >= 1) {
        nowp = 8.0*j*k / (n*n);
        ans += dp(i,j-1,k) * nowp;
        sump += nowp;
    }
    if (k >= 2) {
        nowp = 4.0*k*(k-1) / (n*n);
        ans += dp(i,j,k-1) * nowp;
        sump += nowp;
    }
    if (k >= 1) {
        nowp = 2.0*k / (n*n);
        ans += dp(i,j,k-1) * nowp;
        sump += nowp;
    }
    if (sump > 0) ans /= sump;
    else ans = 0;
    // cerr << i << " " << j << " " << k << ":" << ans <<","<<sump<< "\n";
    return f[i][j][k] = ans;
}

void solve() {
    cin >> n;
    for (int i = 0;i <= (int)n;++i) {
        for (int j = 0;j <= (int)n;++j) {
            for (int k = 0;k <= (int)n;++k) {
                f[i][j][k] = -1;
            }
        }
    }
    cout << fixed << setprecision(10) << dp(n,0,0);
}

signed main() {
    ios::sync_with_stdio(false);
    cin.tie(0);cout.tie(0);
    int T = 1;//cin >> T;
    while (T--) solve();
    return 0;
}
```

