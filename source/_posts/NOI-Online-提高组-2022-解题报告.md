title: NOI Online 提高组 2022 解题报告
categories: 解题报告
tags: [思维,单调栈,树状数组,二维数点]
date: 2022-03-28 12:57:00
---
~~我认为本场 NOI Online 是正确的，客观的，合理的，明晰的，真实的，辩证的，深刻的，通达的，优美的，巧妙的，精辟的，雅正的，机智的，全面的，明白晓畅的，不偏不倚的，恰如其分的，滴水不漏的，不容质疑的，切中要害的，一针见血的，淋漓尽致的，深谙事理的，真知灼见的，发蒙振聩的，微言大义的，金声玉振的，形而上学的，透过现象看本质的，知其然而知其所以然的，可供世人仿效的，千古颠扑不破的。~~


一个出了两个数点题，却没有一个 DP/字符串/数学 的 round。

<!--more-->

# A.丹钓战

> 有 $n$ 个二元组 $(a_i, b_i)$，编号为 $1$ 到 $n$。
>
> 有一个初始为空的栈 $S$，向其中加入元素 $(a_i, b_i)$ 时，先不断弹出栈顶元素直至栈空或栈顶元素 $(a_j , b_j)$ 满足 $a_i \neq a_j$ 且 $b_i < b_j$，然后再将其加入栈中。
>
> 如果一个二元组入栈后栈内只有这一个元素，则称该二元组是“成功的”。
>
> 有 $q$ 个询问 $[l_i, r_i]$，表示若将编号在 $[l_i, r_i]$ 中的二元组按编号从小到大依次入栈，会有多少个二元组是“成功的”。
>
> 询问之间相互独立。
>
> $1 \leq n, q \leq 5 \times 10^5$

## 思路

首先观察一下这个题的性质，记在 $[l,x]$ 内单调栈内只有一个元素 $x$ ，那么 $x$ 必然满足 $\forall y,a_y = a_x \text{ or } b_y \geq b_x$，这个约束条件显然是具有单调性的。

故我们不妨先对所有 $n$ 个元素记录一下这个元素在单调栈中的前驱  $pre_x$ ，这个题的一个重要观察是，对于一个区间 $[l,r]$ 中的所有元素， $pre_x < l$ 的值是关键点。这个观察的原因是，当 $pre_x < l$ 时，说明 $[l,x]$ 的元素必然满足上述提到的条件，否则设 $y \in [l,x]$ 不满足，那么 $pre_x$ 的值就应该为 $y$。

所以我们可以把 $(pre_x,x)$ 作为点，把询问离线下来按照 $pre_x$ 排序，然后将所有第二维小于 $pre_x$ 的节点全部插入树状数组中，每次查询 $[l,r]$ 有多少点即可，当然也可以主席树，总之是个二维数点就对了。

时间复杂度 $\mathcal{O}(m + n \log n)$

## 代码

```cpp
const int N = 5e5 + 10;

int s[N],t[N],n,m,a[N],b[N],top,p[N],ans[N];
pii w[N];

struct Query {
    int l,r,pos;
    friend inline bool operator < (Query &a,Query &b) {
        return a.l < b.l;
    }
}q[N];

int lowbit(int x) {return (x & (-x));}

void modify(int x,int v) {
    for (;x <= n;x += lowbit(x)) t[x] += v;
}

int query(int x) {
    int ans = 0;
    for (;x;x -= lowbit(x)) ans += t[x];
    return ans;
}


signed main() {
    read(n,m);
    for (int i = 1;i <= n;++i) read(a[i]);
    for (int i = 1;i <= n;++i) read(b[i]);
    for (int i = 1;i <= n;++i) {
        while (top && (a[s[top]] == a[i] || b[s[top]] <= b[i])) --top;
        p[i] = s[top];
        s[++top] = i;
        w[i] = mp(p[i],i);
    } 
    sort(w+1,w+1+n);
    for (int i = 1;i <= m;++i) {
        read(q[i].l,q[i].r);q[i].pos = i;
    }
    sort(q+1,q+1+m);
    int now = 1;
    for (int i = 1;i <= m;++i) {
        while (now <= n && w[now].first < q[i].l) modify(w[now].second,1),++now;
        ans[q[i].pos] = query(q[i].r) - query(q[i].l-1);
    }
    for (int i = 1;i <= m;++i) {
        printf("%d\n",ans[i]);
    }
	return 0;
}
```

# B.讨论

> 有 $n$ 个人正在打模拟赛，模拟赛有 $n$ 道题目。  有两人都会的题目并且没有人会的题目包含另一个人时，两者之间才会讨论。  
>
> （定义第 $i$ 个人会的题目的集合为 $S_i$ ，即当 $S_x\cap S_y\neq\varnothing\land S_x\not\subseteq S_y\land S_y\not\subseteq S_x$ 时，第 $x$ 人和第 $y$ 人会讨论）  
>
> 为了让模拟赛的效果更好，希望你可以找出一对会讨论的人或判断不存在。
>
> $m=\sum k_i$，$1\le n\le {10}^6$，$1\le m\le 2\times {10}^6$

## 思路

首先按照集合的大小从大到小排序，依次处理每个集合。

我们可以维护一个数组 $f_i$ 代表当前题目 $i$ 最后被哪个集合包含了。对于当前集合 $S$，我们考虑元素 $x \in S$ ，如果存在不同的 $f_x$ 并且 $S$ 与这个集合不是包含关系，那么我们的问题就解决了。如果不存在的话，考虑包含那么原来储存的那个集合就没有用了，因为后面若交了肯定还是包含，就把原来那个删掉，把当前集合塞进去。

现在就来解决如何查询 $f_x$ 的问题，这部分有两种方法。

第一种方法：

比如 $S_1= \{1, 2, 4\}$ 和 $S_2 = \{3\}$ 就表示成 $f = [1, 1, 2, 1]$。然后判断的时候就可以直接判断了，比如插入了 $\{1, 2, 5\}$，然后发现 $f_1 = 1$，则把 $S_1$ 拿出来，把 $S_1$ 加入桶里，然后统计 $\{1, 2, 5\}$ 中有几个元素在桶中，这里有两个，$2 < 3$ 说明没有完全包含。

每个集合只会被塞入一次，也只会被检查和删除一次，所以复杂度是 $\mathcal{O}(n)$的。

第二种方法：

我们依然采用最上边上面维护 $f_i$ 的方法，但是我们考虑我们插入一个集合的时候，如果这个集合有全新的元素我们是不是就解决了这个问题？那么如何判断？记当前集合 $S$ 集合内为元素 $x$ ：

- 若存在 $f_x = 0$ 则说明这个集合相较之前的所有集合有新元素。
- 若至少存在两个不同的 $f_x$ ，那么相对于题量较少的 $f_x$ ， $S$ 肯定是有新题的。因为对于题量少的那个 $f_x$ ，其必然不会另一道交叉的题。

所以我们只要记录和谁有共同题目，然后最终的答案就是题量较少的那个集合以及当前集合。时间复杂度同第一种方法。

## 代码

这里给出第二种方法的参考实现：

```cpp
const int N = 1e6 + 10;

int n,k[N],c[N],f[N];
vector <int> p[N];

bool cmp(int x,int y) {
    return k[x] > k[y];
}

void solve() {
    read(n);
    memset(f,0,sizeof f);
    for (int i = 1;i <= n;++i) {
        read(k[i]);c[i] = i;
        p[i].clear();
        for (int j = 1;j <= k[i];++j) {
            int x;read(x);
            p[i].push_back(x);
        }
    }
    sort(c+1,c+1+n,cmp);
    bool flag = 1;
    int ans1,ans2;
    for (int i = 1;i <= n;++i) {
        int x = c[i];
        int f1=0,f2=0;
        if (!flag) break;
        for (auto y : p[x]) {
            int las = f[y];f[y] = x;
            if (!las) las = x;
            if (!f1) f1 = las;
            else if (f1 != las) f2 = las;
            if (f1&&f2) {
                if (f1 != x && f2 != x) {
                    //printf("%d %d %d\n",f1,f2,x);
                    if (k[f1] > k[f2]) f1 = x;
                    else f2 = x;
                }
                ans1 = f1,ans2 = f2;
                flag = 0;
                break;
            }
        }
    }
    if (flag) puts("NO");
    else {
        puts("YES");
        printf("%d %d\n",ans1,ans2);
    }
}

signed main() {
    int T;read(T);
    while (T--) solve();
	return 0;
}

```

