---
title: Luogu P1032 字串变换 解题报告
categories: 解题报告
tags: [BFS,搜索,字符串]
date: 2020-02-11 12:02:00
---
> [【NOIP2002】字串变换](<https://www.luogu.com.cn/problem/P1032>)

> 给你两个字符串 $A,B$ ，并给你 $n$ 个规则$(n\leq 6)$ ，求从 $A$ 到 $B$ 最小的变换步数（若$10$步内无法变换则无解，字符串长度不能超过$20$）。

<!--more-->
## 解题思路：

从"$10$步内无法变换则无解" 则可以马上反应到：这是个搜索题（要素察觉）

既然是个搜索题，我们就要确定几点：搜索方式，搜索状态，搜索转移，搜索边界。

方式：这题是求最快的步数，所以我们采用 bfs 显然会比 dfs 快很多，而且有起点和终点状态我们就可以用双向搜索（具体不多说了），这里我们讲单向 bfs 的方法。

状态：首先，题目是对一个字符串进行操作，所以字符串显然是其中的一个状态

其次，题目中提到了对步数的限制，所以步数也是其中的一个状态。

```cpp
struct node {
    int sum;//步数
    string s;//当前字符串
}
```

转移：其实题目中给的已经很明显了：就是按照这几种规则去转移当前字符串即可。

边界：题目中也有明显的指出，每次取出队头节点时要判断一下步数是否$ > 10$。

以上便是此题的基本思路，但是这题的代码及其繁琐，所以我们着重讲一讲代码的实现。

1. 储存规则：规则是两个字符串，我们可以用一个`pair<string,string>`来储存。

2. 转移：我们用的是`string`，在这里，我们可以使用`stirng::find` 和 `string::replace`来实现字串的替换。

3. 特判：每次取出节点的时候判断一下步数，每次变换完判断一下长度是否超限。

   

由此，我们可以写出一份基本的代码：

```cpp
vector<pair<string,string> > ma;//储存变换规则
queue <node> s;
void bfs() {
	node s = (node){a,0};
	q.push(s);//初始状态插入
	while (!q.empty()) {
		node p = q.front();q.pop();
		string now = p.s;	
		int num = p.sum;//取出当前状态
		if (num > 10) continue;//判断当前步数是否>10
		for (int i = 0;i < tot;++i) {//循环判断
			string a1 = ma[i].first,b1 = ma[i].second;
			if (now.find(a1) != std::string::npos) {//如果能找到
				for (size_t j = now.find(a1);j != std::string::npos;j = now.find(a1,j+1)) {
                    //上行代码有几个点要讲解的：1.使用size_t是因为string::npos是size_t的，如果不懂什么意思可以把它当作unsigned long long(有大佬说系统位数是32位这个值会变成unsigned int)来看待 2.前面一个find是找到最开始串里有没有这个最初的串，后面是在这个变换完的串之后看是否还有可以变换的
					string t = now;
					t.replace(j,a1.length(),b1);//将可以变换的变换为 b1
					if (t == b) {cout << num+1;return ;}//如果找到了，可以直接退出
					if (t.length() <= 20) {//判断字符串长度
                        node pp = (node){t,num+1};
                        q.push(pp);
                        se.insert(t);
					}
				}
			}
		}
	}
	cout << "NO ANSWER!";
}
```

这份代码没什么问题，但是我们可以分析一下这个题目性质：

显然，一个串可以被几个不同的规则搜到，但是由于我们是 bfs 所以可以证明最开始搜到的那个情况是最优的，之后搜到的情况都可以直接舍弃。

所以，我们可以把用过的情况存在一个 `map ` 里，如果这个情况搜过了就跳过。



```cpp
if (t.length() <= 20) {
    if (se.find(t)==se.end()){
        node pp = (node){t,num+1};
        q.push(pp);
        se.insert(t);
    }
}
```

这样，你的代码就没什么问题了。

## 代码：

```cpp
#include <iostream>
#include <cstring>
#include <string>
#include <cmath>
#include <queue>
#include <map>
#include <set>
using namespace std;

#define ll long long
#define ri register int

const int N = 100001;
const int M = 500001;

struct node {
	string s;int sum;
};
vector<pair<string,string> > ma;
string a,b,a1,b1;
int ans,cnt,tot;

set <string> se;
queue <node> q;

void bfs() {
	node s = (node){a,0};
	q.push(s);
	while (!q.empty()) {
		node p = q.front();q.pop();
		string now = p.s;	
		int num = p.sum;
		if (num > 10) continue;
		for (int i = 0;i < tot;++i) {
			string a1 = ma[i].first,b1 = ma[i].second;
			if (now.find(a1) != std::string::npos) {
				for (size_t j = now.find(a1);j != std::string::npos;j = now.find(a1,j+1)) {
					string t = now;
					t.replace(j,a1.length(),b1);
					if (t == b) {cout<<num+1;return ;}
					if (t.length() <= 20) {
						if (se.find(t)==se.end()){
							node pp = (node){t,num+1};
							q.push(pp);
							se.insert(t);
						}
					}
				}
			}
		}
	}
	cout << "NO ANSWER!";
}

int main() {
	ios::sync_with_stdio(false);
	cin.tie(0);cout.tie(0);
	cin >> a >> b;
	while (cin >> a1 >> b1) {
		ma.push_back(make_pair(a1,b1));
		++tot;
	}
	bfs();
	return 0;
}
```

