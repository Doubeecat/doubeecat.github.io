---
title: ACWing 95 费解的开关 解题报告
categories: 解题报告
tags: [递归,位运算]
date: 2020-04-07 14:07:00
---


> [费解的开关](<https://www.acwing.com/problem/content/97/>)
>
> 给你一个 $5\times 5$ 的方格，每次操作可以改变上下左右中五个格子，给定始状态，判断是否可能在6步以内使所有的灯都变亮。

<!--more-->

## 解题思路：

这道题目首先看，我们就知道必然与位运算有着密切的关系，因为出现了0和1，这是一个重要的发现.

接着我们在仔细分析题意，我们知道如果纯暴力枚举的话，必然是会超时的，那么如何优化呢？

我们需要从题目中找出非常有用的性质来优化，这是一个大致的思路方向。

每一个位置顶多只会操作一次。因为如果操作两次的话，相当于不操作，必然是不满足最优解。

在一套方案中，操作的顺序无关紧要，这一个略加思索便可得知。

最重要的性质，如果我们确定了第I行的操作方案的话，那么后面的行数都可以依此递推，下面给出一个详细的解答。

```
11011
10110
01111
11111
```
比如说这个例子，如果我们确定了第1行，那么第二行所有的0(坐标：$a_{i,j}$)  

都只能是第三行 $a_{i+1,j}$ 来修改了，因为如果你第二行修改的话，那么第一行将会打乱，下面每一行依此类推。

## 代码：

```cpp

char a[N][N];
const int dx[5] = {0,0,1,0,-1},dy[5] = {0,1,0,-1,0};

void change(int x,int y) {
	for (int i = 0;i < 5;++i) {
		int ax = x + dx[i],ay = y + dy[i];
		if (ax < 0 || ax > 4 || ay < 0 || ay > 4) continue;
		a[ax][ay] ^= 1;
	}
	return ;
}

bool check() {
	bool success = 1;
	for (int i = 0;i < 5;++i) {
		if (a[4][i] == '0') {
			success = 0;
		}
	}
	return success;
}

int work() {
	int ans = 233333333;
	for (int k = 0;k < 1<<5;++k) {
		int step = 0;
		char b[N][N];
		memcpy(b,a,sizeof(a));
		for (int i = 0;i < 5;++i) {
			if (k >> i & 1) {
				step++;
				change(0,i);
			}
		}
		for (int i = 0;i < 4;++i) {
			for (int j = 0;j < 5;++j) {
				if (a[i][j] == '0') {
					step++;
					change(i+1,j);
				}
			}
		}
		if (check()) {
			ans = min(ans,step);
		}
		memcpy(a,b,sizeof(b));
	}
	if (ans <= 6) return ans;
	else return -1;
}

signed main() {
	int T = read();
	while (T -- ) {
		for (int i = 0;i < 5;++i) cin >> a[i];
		printf("%d\n",work());
	}
	return 0;
}
```