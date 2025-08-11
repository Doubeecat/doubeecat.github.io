title: CF GYM101190E Expect to Wait 解题报告
categories: 解题报告
tags: [扫描线]
date: 2021-06-29 02:52:00
---
> 有一个独轮车租借中心，在一天之内有 $n$ 次事件，每次事件为若干个人在某一时刻来借车或者还车。
> 你不知道初始中心有几辆车，因此有 $q$ 组询问，每次给出中心初始的车的数量，问所有人的最小总等待时间。
> $n,q\leq 10^5$

<!--more-->

我们以时间为  $x$ 轴，当前等待为 $y$ 轴，那么借车就变成了加，还车就变成了减，每次询问的话实际上是做一条平行于 $x$ 轴的直线，让你求直线上曲线和这条直线围成的面积，看图理解比较合理

![](https://i.loli.net/2021/06/30/B4QsUDoRCXz6v3b.png)

![](https://i.loli.net/2021/06/30/B4QsUDoRCXz6v3b.png)

也就是让你求出橙色部分（对于下面那条直线来说）

![](https://i.loli.net/2021/06/30/Ext1iHIrYk69svD.png)

那么这题就被我们转化为了一个扫描线问题。具体怎么写还是要思考一下的，~~因为之前没写过~~

首先将直线按纵坐标从大到小排序，我们设两条直线 $l_1 = a,l_2 = b,(a > b)$

对于直线和线段之间的部分，我们也是把线段从大到小排序，这样我们处理过来就是单调的。设线段所在直线交 $y$ 轴于 $c$,长度为 $d$ 那么一条线段和这条直线围成的面积就是 $(c-a) \times d$

对于两条直线之间的部分，如图绿色部分是需要加上的，红色部分是上一条计算过的，橙色部分是我们这条直线会计算到的

![](https://i.loli.net/2021/06/30/spE6kbSmVZ1hAdn.png)

那么计算绿色部分也很简单，我们记录一下前面处理到的几条线段总长度，乘上 $a-b$ 即可，线段不会产生交集，所以这样是对的。

时间复杂度 $O(n\log n)$,主要就是在于排序

Code:

```cpp
#include <algorithm>
#include <iostream>
#include <cstring>
#include <cstdio>
#include <cctype>
#include <vector>
#include <cmath>
using namespace std;
#define ll long long
#define ri int
#define int long long
char buf[1 << 20], *p1, *p2;
#define getchar() (p1 == p2 && (p2 = (p1 = buf) + fread(buf, 1, 1 << 20, stdin), p1 == p2)?EOF: *p1++)
template <typename T> inline void read(T &t) {
	ri v = getchar();T f = 1;t = 0;
	while (!isdigit(v)) {if (v == '-')f = -1;v = getchar();}
	while (isdigit(v)) {t = t * 10 + v - 48;v = getchar();}
	t *= f;
}
template <typename T,typename... Args> inline void read(T &t,Args&... args) {
	read(t);read(args...);
}

const int N = 1e6 + 10;

struct line {
    int len,k,bel;
}a[N];

struct query {
    int q,bel;
}p[N];

bool cmp1(line a1,line a2) {
    return a1.k > a2.k;
}

bool cmp2(query a,query b) {
    return a.q > b.q;
}

ll n,m,tot,t[N],k[N],ans[N];

signed main() {
    freopen("expect.in","r",stdin);
    freopen("expect.out","w",stdout);
    read(n,m);
    for (int i = 1;i <= n;++i) {
        char opt = getchar();
        read(t[i],k[i]);
        if (opt == '+') k[i] *= -1ll;
    }

    int las = k[1];
    for (int i = 2;i <= n;++i) {
        a[++tot].len = t[i] - t[i-1];
        a[tot].k = las;
        las += k[i];
    }
    las = max(0ll,las);
    sort(a+1,a+1+tot,cmp1);
    for (int i = 1;i <= m;++i) {
        read(p[i].q);p[i].bel = i;
    }
    sort(p+1,p+1+m,cmp2);
    ll tmp = 0,pos = 1,len = 0;
    for (int i = 1;i <= m;++i) {
        if (p[i].q < las) {
            ans[p[i].bel] = -1;
            continue;
        }
        tmp += (p[i-1].q - p[i].q) * len;
        while (pos <= tot && a[pos].k > p[i].q) {
            tmp += (a[pos].k - p[i].q) * a[pos].len;
            len += a[pos].len;
            ++pos;
        }
        ans[p[i].bel] = tmp;
    }
    for (int i = 1;i <= m;++i) {
        if (ans[i] == -1) puts("INFINITY");
        else printf("%lld\n",ans[i]);
    }
	return 0;
}
```