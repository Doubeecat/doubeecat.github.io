title: CF363D Renting Bikes 解题报告
categories: 解题报告
tags: [贪心,二分]
date: 2020-10-01 15:59:00
---
> [CF363D Renting Bikes](https://www.luogu.com.cn/problem/CF363D)
>
> 模拟赛补题计划 ON
>
> 有 $n$ 个学生要租车，一共有 $m$ 辆车，每辆车有一个价钱 $p_i$，每个学生有自己的钱 $b_i$，并且他们的钱**只能自己用**，每个人只能租一辆车。他们有 $a$ 公用的钱，求出
> 
> 1. 最多有多少个人能租到车
> 
> 2. 在保证尽量多的人租到车的前提下，每个人出的自己的钱总和最小是多少。
>
> $n \leq 10^5$
<!--more-->

## 解题思路：

二分 + 贪心

这两个问题其实可以分开来解决，我们首先考虑第一个问题。

记最多有 $x$ 个人可以租到车，显然这个 $x$ 满足单调性，所以考虑二分解决。

现在问题转化成了，如何判断是否能让 $x$ 个人租到车？

这里的问题看上去范围就不是很好 DP，所以我们考虑贪心。

一个错误的贪心思路是：先尽量用班级的钱，花到不能买下整个时再去拼凑后面的车。样例 2 就可以 hack 掉。

然而，我们依然可以从这里得到一些对我们有意义的东西。**拼凑**一辆车是一个非常重要的思路，为了尽量多，我们要对于每辆车思考一下如何拼凑。

显然，我们选择的 $x$ 辆车必定是最便宜的 $x$ 辆车，这个证明十分简单，~~当然也可以感性理解~~。

接下来就是考虑人的问题了，因为每个人对于整个答案的贡献都是 1，所以哪个人选哪个车对答案是没影响的。同理，我们也想一下如何让人选车最优。

一个不那么显然但是正确的方案：让钱最多的 $x$ 个人，按照从少到多的顺序来买车。

证明：

我们有 $k$ 个男孩 $a_1 \dots a_k$ 和 $k$ 个自行车 $b_1 \dots b_k$，是否有其他方案让我们用更少的钱。

取一组 $i,j(i < j)$，令 $a_i$ 买 $b_i$， $a_j$ 买 $b_j$ 然后讨论是否需要交换他们（也就是$a_j$ 买 $b_i$， $a_i$ 买 $b_j$）

情况1. 若 $a_i \geq b_i$，则有 $a_i,a_j \geq b_i$,所以这里肯定用 $a_i$ 买 $b_i$，$a_j$ 买 $a_j$ 最好，不用交换。

情况2. 若 $a_i < b_i$，则不论 $b_i > a_j$ 还是 $b_i \leq a_j$，交换都不会让结果变得更好，所以不用交换。

所以，我们可以用这个结论写出 check

```cpp

bool check(int x) {
    int nk = k;//当前剩下的通用钱数
    int pos = x;//从第x小到最小枚举
    for (int i = 1;i <= x;++i) {
        if (b[i] < p[pos--]) {//如果钱不够，就用通用钱去填
            nk = nk + b[i] - p[pos+1] ;
        }
        if (nk < 0) return 0;//如果发现坑太大，填不上了，则说明 x 不行
    }
    //if (nk >= 0) printf("ans:%d\n",x);
    return nk >= 0 ? 1 : 0 ;//最后记得再判断一次
}

```

对于第二问，从第一问的贪心中我们可以发现我们怎么选择的并不重要，只要考虑最后的结果就行，

记选择车辆的总价钱为 $p_{total}$ , 答案就是 $p_{total} - k$

## 代码：

```cpp
inline bool cmp(int x,int y) {return x > y;}

bool check(int x) {
    int nk = k,pos = x; 
    for (int i = 1;i <= x;++i) {
        if (b[i] < p[pos--]) {
            nk = nk + b[i] - p[pos+1] ;
        }
        if (nk < 0) return 0;
    }
    return nk >= 0 ? 1 : 0 ;
}

int main() {
    read(n,m,k);
    for (int i = 1;i <= n;++i) read(b[i]);
    for (int i = 1;i <= m;++i) read(p[i]);
    sort(b+1,b+1+n,cmp);sort(p+1,p+1+m);
    int l = 0,r = min(n,m),ans = 0;
    while (l <= r) {
        int mid = (l + r) >> 1;
        if (check(mid)) ans = mid,l = mid + 1;
        else r = mid - 1;
    }
    int sum = 0;
    for (int i = 1;i <= ans;++i) sum += p[i];
    printf("%d %d",ans,max(0,sum - k));
    return 0;
}
```


