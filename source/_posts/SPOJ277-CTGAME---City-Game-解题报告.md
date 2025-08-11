title: SPOJ277 CTGAME - City Game 解题报告
categories: 解题报告
tags: [思维,单调栈]
date: 2020-10-26 13:45:08
---

> [SP277 CTGAME - City Game](https://www.luogu.com.cn/problem/SP277)
> 
> 有 $k$ 片土地，每片土地被分成 $n \times m$ 个格子，每个格子里写着 `R` 或者 `F`.
> 
> 对于每片土地，找一块矩形土地，要求这片土地都标着 `F`并且面积最大。
> 
> $n,m \leq 10^3$

<!--more-->

## 解题思路

单调栈/悬线法

类似这类有障碍点找最大矩形的题显然可以用悬线法来做，不过这题里是大材小用。

其实这题是[这题](https://www.luogu.com.cn/problem/SP1805)的变式，一时没有思路可以看下上面那题。

考虑怎么构成一个极大矩形，发现一个矩形实际上可以按照行拆分成一条一条的，比如这样。

```
FFF    F F F
FFF => F F F
FFF    F F F
       1 2 3
```

所以这就启发我们，对于每一个格子，可以先将其上边连续的 `F` 数量处理出来，记作 $h_{i,j}$，这里可以采用递推。

若 $a_{i,j} = F$ ，则 $h_{i,j} = h_{i-1,j} + 1$ 

否则 $h_{i,j} = 0$ 

我们在读入时预处理就能做到。

```cpp
cin >> n >> m;
    for (int i = 1;i <= n;++i) {
        for (int j = 1;j <= m;++j) {
            cin >> g[i][j];
            if (g[i][j] == 'F') h[i][j] = h[i-1][j] + 1;
        }
    }
```

这样，我们就把问题拆分成了每一行上单独的[这题](https://www.luogu.com.cn/problem/SP1805)，我们对于每一行跑一遍单调栈就解决了。

具体地说，我们现在对于一堆矩形，要求出其最大面积。我们建立两个栈，一个用来表示高度，一个用来表示宽度。从左到右扫描每个矩形：
- 若当前矩形比栈顶矩形的高度高，直接进栈。
- 反之，不断取出栈顶直到当前矩形比栈顶矩形的高度大。注意在出栈过程中，这些矩形也是有用的。我们考虑取他们的公共部分，也就是累加他们的宽度之和，并且用每个矩形的高度乘上累计宽度取更新答案。在这个过程结束后，我们就把当前高度与累计宽度的矩形再进栈。

整个过程结束之后，我们要将栈排空，这里我们可以直接添加一个高度为 $0$ 的矩形方便实现，写成代码也就是

```cpp
stack <int> s1,s2;

int work(int x) {
    int ans = 0;
    while (!s1.empty()) s1.pop();//清空两个栈
    while (!s2.empty()) s2.pop();
    s1.push(0);s2.push(0);//保证不访问空栈
    for (int i = 1;i <= m + 1;++i) {
        int now = h[x][i];//当前位置最大的 F 连续高度
        if (now > s1.top()) {s1.push(now),s2.push(1);}//直接进栈的情况
        else {
            int wid = 0;
            while (!s1.empty() && now < s1.top()) {
                wid += s2.top();//累加高度
                ans = max(ans,wid * s1.top();//更新答案
                s1.pop(),s2.pop();
            }
            s1.push(now),s2.push(wid+1);//最后入栈
        }
    }
    return ans;
}
```

对于每一行，我们都执行一个这样的过程

```cpp
int ans = 0;
for (int i = 1;i <= n;++i) ans = max(ans,work(i));    
```

时间复杂度 $O(nm)$

## 代码

```cpp
#include <cstring>
#include <iostream>
#include <cstdio>
#include <cctype>
#include <cmath>
#include <stack>
#include <algorithm>
using namespace std;
#define ll long long
#define ri register int

char buf[1 << 20], *p1, *p2;
//#define getchar() (p1 == p2 && (p2 = (p1 = buf) + fread(buf, 1, 1 << 20, stdin), p1 == p2)?EOF: *p1++)
inline int read() {
	ri v = getchar();int f = 1,t = 0;
	while (!isdigit(v)) {if (v == '-')f = -1;v = getchar();}
	while (isdigit(v)) {t = t * 10 + v - 48;v = getchar();}
	return t *= f;
}

const int N = 1e3+10;

int n,m,h[N][N];

char g[N][N];

stack <int> s1,s2;

int work(int x) {
    int ans = 0;
    while (!s1.empty()) s1.pop();
    while (!s2.empty()) s2.pop();
    s1.push(0);s2.push(0);
    for (int i = 1;i <= m + 1;++i) {
        int now = h[x][i];
        if (now > s1.top()) {s1.push(now),s2.push(1);}
        else {
            int wid = 0;
            while (!s1.empty() && now < s1.top()) {
                wid += s2.top();
                ans = max(ans,wid * s1.top());
                s1.pop(),s2.pop();
            }
            s1.push(now),s2.push(wid+1);
        }
    }
    return ans;
}

void solve() {
    memset(h,0,sizeof h);
    cin >> n >> m;
    for (int i = 1;i <= n;++i) {
        for (int j = 1;j <= m;++j) {
            cin >> g[i][j];
            if (g[i][j] == 'F') h[i][j] = h[i-1][j] + 1;
        }
    }
    int ans = 0;
    for (int i = 1;i <= n;++i) ans = max(ans,work(i));
    printf("%d\n",ans * 3);
}

signed main() {
    int T;cin >> T;
    while (T--) solve();
	return 0;
}
```