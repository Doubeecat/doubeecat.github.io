---
title: CF1648A Weird Sum 解题报告
categories: 解题报告
tags: []
date: 2022-07-20 02:49:00
---
# CF1648A. Weird Sum

> 给出两个整数 $n,m$，和一个 $n \times m$ 的矩阵。若该矩阵中两元素相同，$sum$ 就加上它们所在位置的曼哈顿距离。求 $sum$ 的值。
> 
>  $1 \leq n \le m, n \cdot m \leq 100000$ 

<!--more-->

## 思路

首先给出曼哈顿距离的定义式：

$$
dis_{i,j} = |x_i - x_j| + |y_i - y_j|
$$

是两个绝对值的加和，而题目中所让我们求的同种颜色的两两之间曼哈顿距离，可以表示为如下式：

$$
\sum\limits_{col_{x_i,y_i} = k} \sum\limits_{col_{x_j,y_j} = k,i\not ={j}} |x_i - x_j| + |y_i - y_j| 
$$

显然，我们可以把 $x,y$ 拆开来计算，以下先考虑 $x$ 的部分如何计算，$y$ 的部分同理。

首先，我们可以把 $x_i$ 进行排序，记当前颜色有 $p$ 个点，那么 $x$ 部分的贡献可以如下表示

$$
\begin{aligned}
    &\sum\limits_{i =1}^p \sum\limits_{j = 1}^{i-1} |x_j - x_i| \\
    = &\sum\limits_{i =1}^p \sum\limits_{j = 1}^{i-1} x_i - x_j\\
    = &\sum\limits_{i =1}^p (i-1)\times x_i - \sum\limits_{j = 1}^{i-1} x_j
\end{aligned}
$$

显然，这个式子里的 $(i-1)\times x_i$ 我们可以在 $\mathcal{O}(1)$ 的时间复杂度内算出，而 $\sum\limits_{j = 1}^{i-1} x_j$ 可以在一个维护前缀和的过程中算出，时间复杂度为 $\mathcal{O}(p)$.

对每种颜色做一遍上面的计算，总时间复杂度为 $\mathcal{O}(\sum p ) = \mathcal{O}(n\times m)$，可以通过本题。$y$ 的计算同理，此处不再赘述。

## 代码

```cpp
const int N = 1e5 + 10;

int n,m;
vector <int> c1[N],c2[N];

void solve() {
    for (int i = 1;i <= 100000;++i) c1[i].clear(),c2[i].clear();
    read(n,m);
    for (int i = 0;i < n;++i) {
        for (int j = 0;j < m;++j) {
            int col;read(col);
            c1[col].push_back(i);
            c2[col].push_back(j);
        }
    }
    int ans = 0;
    for (int i = 1;i <= 100000;++i) {
        if (c1[i].empty()) continue;
        sort(c1[i].begin(),c1[i].end());
        int pre = 0;
        for (int j = 0;j < c1[i].size();++j) {
            ans += j * c1[i][j] - pre;
            pre += c1[i][j];
        }
    }
    for (int i = 1;i <= 100000;++i) {
        if (c2[i].empty()) continue;
        sort(c2[i].begin(),c2[i].end());
        int pre = 0;
        for (int j = 0;j < c2[i].size();++j) {
            ans += j * c2[i][j] - pre;
            pre += c2[i][j];
        }
    }

    printf("%lld\n",ans);
}

signed main() {
    solve();
	return 0;
}
```