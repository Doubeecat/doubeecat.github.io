---
title: UVA1560 Extended Lights Out 解题报告
categories: 解题报告
tags: [暴力,枚举,位运算]
date: 2020-08-11 08:05:48
---

> [UVA1560 Extended Lights Out](https://www.luogu.com.cn/problem/UVA1560)
>
> 有一个由按钮组成的矩阵，其中每行有6个按钮，共5行。每个按钮的位置上有一盏灯。当按下一个按钮后，该按钮以及周围位置(上边、下边、左边、右边)的灯都会改变一次。
> 
> 即，如果灯原来是点亮的，就会被熄灭；如果灯原来是熄灭的，则会被点亮。在矩阵角上的按钮改变3盏灯的状态；在矩阵边上的按钮改变4盏灯的状态；其他的按钮改变5盏灯的状态。
>
> 请你写一个程序，确定需要按下哪些按钮，恰好使得所有的灯都熄灭。
> 
> 根据上面的规则，我们知道
> 
> 1. 第2次按下同一个按钮时，将抵消第1次按下时所产生的结果。因此，每个按钮最多只需要按下一次
> 
> 2. 各个按钮被按下的顺序对最终的结果没有影响
> 
> 3. 对第1行中每盏点亮的灯，按下第2行对应的按钮，就可以熄灭第1行的全部灯。
> 
> 如此重复下去，可以熄灭第1、2、3、4行的全部灯。同样，按下第1、2、3、4、5列的按钮，可以熄灭前5列的灯。

<!--more-->
### 思路：

暴力模拟

这题的题面已经讲得很清楚了，我们分部分来考虑。

1. 按下按钮

这里用了两个常量数组 `dx,dy`  来表示修改的五个位置。

因为每个位置只有 $0,1$ 两个状态，所以直接异或。

```cpp
const int dx[5] = {0,0,1,0,-1},dy[5] = {0,1,0,-1,0};

void change(int x,int y) {
    for (int i = 0;i < 5;++i) {
        int ax = x + dx[i],ay = y + dy[i];
        if (ax < 0 || ax > 4 || ay < 0 || ay > 5) continue;
        a[ax][ay] ^= 1;
    }
    return ;
}
```

2. 枚举点击方案

这里我们可以枚举第一行的点击方案，从第一行推出接下来的行如何点击。

这里用位运算可以很方便实现，我们把二进制意义的第 $i$ 位如果是 $1$ 则表示为按下，如果是 $0$ 则不动。

每次点击上一行为 `1` 的地方，保证操作完上一行全部都没点亮。

建议配合代码食用。

```cpp
for (int i = 0;i < 6;++i) {
    if (k & (1 << i)) {
        change(0,i);
        pos[0][i] = 1;
    }
}//枚举第一行方案
for (int i = 1;i < 5;++i) {
    for (int j = 0;j < 6;++j) {
        if (a[i-1][j]) {
            pos[i][j] = 1;
            change(i,j);
        }
    }
}//根据第一行方案递推
```
3. 检查方案是否合法

根据 2，我们可以直接检查最后一行是否全部是 0 ，如果是输出方案即可。

注意 UVA 对输出格式要求很严格 不能有多余的空格

```cpp
bool check() {
    bool success = 1;
    for (int i = 0;i < 6;++i) {
        if (a[4][i]) {
            success = 0;
        }
    }
    return success;
}
if (check()) {
    for (int i = 0;i < 5;++i) {
        for (int j = 0;j < 5;++j) {
            printf("%d ", pos[i][j]);
        }
        printf("%d\n",pos[i][5]);
    }
    return ;
}
```

下面给出全部代码

### 代码：
```cpp
#include <cstring>
#include <cstdio>
#include <cctype>
#include <cmath>
#include <iostream>
using namespace std;
#define ll long long
#define ri register int

inline int read() {
    char v = getchar();int x = 0,f = 1;
    while (!isdigit(v)) {if (v == '-')f = -1;v = getchar();}
    while (isdigit(v)) {x = x * 10 + v - 48;v = getchar();}
    return x * f;
}
const int N = 10;
const int INF = 0x3f3f3f3f;

template <typename T> inline T max(T x,T y) {return x>y?x:y;}

int a[N][N];
const int dx[5] = {0,0,1,0,-1},dy[5] = {0,1,0,-1,0};

void change(int x,int y) {
    for (int i = 0;i < 5;++i) {
        int ax = x + dx[i],ay = y + dy[i];
        if (ax < 0 || ax > 4 || ay < 0 || ay > 5) continue;
        a[ax][ay] ^= 1;
    }
    return ;
}

bool check() {
    bool success = 1;
    for (int i = 0;i < 6;++i) {
        if (a[4][i]) {
            success = 0;
        }
    }
    return success;
}

void work() {
    for (int k = 0;k < (1<<6);++k) {
        int b[N][N],pos[N][N];
        memset(pos,0,sizeof pos);
        memcpy(b,a,sizeof(a));
        for (int i = 0;i < 6;++i) {
            if (k & (1 << i)) {
                change(0,i);
                pos[0][i] = 1;
            }
        }
        for (int i = 1;i < 5;++i) {
            for (int j = 0;j < 6;++j) {
                if (a[i-1][j]) {
                    pos[i][j] = 1;
                    change(i,j);
                }
            }
        }
        if (check()) {
            for (int i = 0;i < 5;++i) {
                for (int j = 0;j < 5;++j) {
                    printf("%d ", pos[i][j]);
                }
                printf("%d\n",pos[i][5]);
            }
            return ;
        }
        memcpy(a,b,sizeof(b));
    }
    return ;
}

signed main() {
    int T = read();
    for(int q = 1;q <= T;++q) {
        for (int i = 0;i < 5;++i) {
            for (int j = 0;j < 6;++j) {
                cin >> a[i][j];
            }
        }
        printf("PUZZLE #%d\n",q);
        work();
    }
    return 0;
}
```