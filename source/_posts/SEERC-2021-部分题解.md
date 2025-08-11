---
title: SEERC 2021 部分题解
categories: 解题报告
tags: []
date: 2025-03-31 13:25:00
---
I can do it with a broken heart.

<!--more-->

# A

发现做法和题解不太一样所以写一下，我们考虑字典序比较，那么显然从第一位开始看起。我们统计所有满足 $s_i < t_i$ 的 $i$，那么对于 $1 \leq j < i$，我们要找的是最长的一个后缀满足 $s_j = t_j$，记这样的后缀长度为 $len$，这样的答案就是 $(len + 1) \times (n-i)$ 。预处理一下 $len$，时间复杂度为 $O(n)$

```cpp
const int N = 2e5 + 10;
ll diff[N];

void solve() {
    int n;cin >> n;
    string s,t;cin >> s >> t;
    diff[0] = (s[0] == t[0]);  
    for (int i = 1;i < n;++i) {
        if (s[i] == t[i]) {
            diff[i] = diff[i-1] + 1;
        } else {
            diff[i] = 0;
        }
    }
    ll ans = 0;
    if (s[0] < t[0]) ans += n;
    for (int i = 1;i < n;++i) {
        // cerr << diff[i-1] << " " << n-i << "\n";
        if (s[i] < t[i]) {
            ans += (diff[i-1] + 1) * (n-i);
        }
    }
    cout << ans << "\n";

}
```

# C

超级经典的树形背包模型。对于这种绝对众数，我们考虑的是枚举每个数，然后统计出现次数，把相同的记为 $+1$，不相同的记为 $-1$。这样的话就考虑 $f_{x,i}$ 是以 $x$ 为根的子树里和为 $i$ 的方案做树形背包。

注意转移的时候，我们的上下界设为 $cnt_i$，也就是 $i$ 在树上的出现次数。时间复杂度实际上是 $O(n\times \sum cnt_i)$ 的。因为有 $\sum cnt_i = n$，所以这个转移实现正确的复杂度是 $O(n^2)$ 的。

```cpp
const int N = 3e3 + 10;
const ll mod = 998244353;
int c[N],n,cnt[N],siz[N];
vector <int> G[N];
ll ans,f[N][N<<1],tmp[N<<1];
void dfs(int x,int fa,int col) {
    int del = cnt[col];
    siz[x] = 1;
    if (c[x] == col) f[x][del + 1] = 1;
    else f[x][del - 1] = 1;
    for (auto y : G[x]) {
        if (y == fa) continue;
        dfs(y,x,col);
        for (int i = 0;i <= 2 * del;++i) tmp[i] = 0;
        for (int i = min(siz[x],del);i >= -min(siz[x],del);--i) {
            for (int j = min(siz[y],del);j >= -min(siz[y],del);--j) {
                if ((i + j ) <= del && (i + j ) >= -del) {
                    tmp[i+j +  del] = (tmp[i+j+  del] + (f[x][i+del] * f[y][j+del]) % mod)  % mod;
                }
            }
        }
        for (int i = -del;i <= del;++i) {
            f[x][i + del] = tmp[i + del];
        }
        siz[x] += siz[y];
    }
    f[x][del]++;
    for (int i = del + 1;i <= 2*del;++i) ans = (ans + f[x][i]) % mod;
}
void solve() {
    cin >> n;
    for (int i = 1;i <= n;++i) cin >> c[i],cnt[c[i]]++;
    for (int i = 1;i < n;++i) {
        int x,y;cin >> x >> y;
        G[x].push_back(y);
        G[y].push_back(x);
    }
    for (int c = 1;c <= n;++c) {
        for (int x = 1;x <= n;++x){
            for (int i = 0;i <= 2 * cnt[c];++i) {
                f[x][i] = 0;
            }
        }
        
        dfs(1,0,c);
        
        // for (int x = 1;x <= n;++x){
        //     cout << "x:"<<x;
        //     for (int i = 0;i <= 2 * cnt[c];++i) {
        //         cout << "i:"<<i-cnt[c]<<":"<<f[x][i] <<  " ";
        //     }
        //     cout << "\n";
        // }
    }
    cout << ans;
}

```

# F

最大难度其实在读题……根据贪心，关注到我们对于一个恢复药水，早释放毒素显然是更好的。那么就是对于所有有恢复药水的点，我们肯定直接贪心使用毒素，贡献是 $(n-i+1)\times (r+p)$，对于一个平凡的点，贡献是 $(n-i+1)\times p$。排序后模拟一遍，时间复杂度 $O(n\log n)$

```cpp
const int N = 1e6  + 10;
ll n,x,r,p,k;
int sta[N],chos[N];
void solve() {
    cin >> n >> x >> r >> p >> k;
    for (int i = 1;i <= n;++i) {
        char c;cin >> c;
        sta[i] = c - '0';
    }
    vector <pll> vec;
    for (int i = 1;i <= n;++i) {
        if (sta[i]) vec.push_back(mp((n-i+1) * (r+p),i));
        else vec.push_back(mp((n-i+1) * p,i));
    }
    sort(vec.begin(),vec.end());
    reverse(vec.begin(),vec.end());
    for (int i = 0;i < k;++i) {
        chos[vec[i].second] = 1; 
        // cerr << vec[i].first << " " << vec[i].second << "\n";
    }
    ll R = 0,P = 0;
    ll ans = 0;
    for (int i = 1;i <= n;++i) {
        if (sta[i]) ++R;
        if (chos[i]) {
            ++P;
            if (sta[i]) --R;
        }
        ans += x + P*p-R*r;
    }
    cout << ans;
}
```

# G

有点诈骗题了。考虑 $a,b$ 的位置实际上是广义对称的，我们可以把 $a,b$ 分别排序，并且保证 $a_i > b_i$，然后公式就等价于： $a_i - b_j > a_j - b_i$ 转化成 $a_i + b_i > a_j + b_j$ 。也就是按照 $a_i + b_i$ 排序之后，取前面的 $n$ 个 $a_i$，和后面的 $n$ 个 $b_i$ 来算即可。

```cpp
const int N = 2e5 + 10;
pll p[N];
int n;
bool cmp(pll a,pll b) {
    return a.first + a.second < b.first + b.second;
}
void solve() {
    cin >> n;
    for (int i = 1;i <= 2*n;++i) {
        cin >> p[i].first >> p[i].second;
        if (p[i].first > p[i].second) swap(p[i].first,p[i].second);
    }
    sort(p+1,p+1+n*2,cmp);
    ll ans = 0;
    for (int i = 1;i <= n;++i) {
        ans += p[i+n].second - p[i].first;
    }
    cout << ans;
}
```

# J

我们把这个问题看成一个括号匹配问题，记 $cnt_x$ 为字母 $x$ 的个数。那么显然 $A$ 只能做左括号，$C$ 只能做右括号，$B$ 既可以当左括号又可以当右括号。我们先贪心考虑把所有 $B$ 当成左括号，然后遇到不行的再改成右括号。

```cpp
const int N = 2e5 + 10;
int n,m;
bool mkd[N],vis[N];
char s[N];
void solve() {
    cin >> n;m=n;n <<= 1;
    for (int i = 1;i <= n;++i) cin >> s[i];
    int cnt[3] = {0,0,0},tot = 0;
    for (int i = n;i >= 1;--i) {
        if (s[i] == 'C') {
            mkd[i] = 1;++tot;
        } else if (s[i] == 'B') {
            if (!cnt[2]) mkd[i] = 1,++tot;
        } else {
            if (!cnt[2] && !cnt[1]) mkd[i] = 1,++tot;
        }
        ++cnt[s[i] - 'A'];
    }
    int res = m - cnt[0];
    if (cnt[0] > m || cnt[2] > m || res < 0) {
        cout << "NO\n";
        return ;
    }
    //ABBCBC
    //顺序扫一遍   
    //先把C干掉吧，那么就是所有AB在一个队列里，用掉了就vis=1 
    queue <int> qA,qB,q2;
    vector <pii> ans;
    bool flag = 0;

    for (int i = 1;i <= n;++i) {
        if (s[i] == 'C') {
            if (qA.empty() && qB.empty()) {
                flag = 1;
                break;
            }
            if (!qB.empty()) {
                ans.push_back(mp(qB.front(),i));
                vis[i] = vis[qB.front()] = 1;
                qB.pop();
            } else {
                ans.push_back(mp(qA.front(),i));
                vis[i] = vis[qA.front()] = 1;
                qA.pop();
            }
        }
        if (s[i] == 'B') {
            if (res) {
                qB.push(i);
                --res;
            } else {
                if (!qA.empty()){
                    ans.push_back(mp(qA.front(),i));
                    vis[i] = vis[qA.front()] = 1;
                    qA.pop();
                } else {
                    flag = 1;
                    break;
                }
            }
        }
        if (s[i] == 'A') {
            qA.push(i);
        }
    }
    if (flag) {
        cout << "NO";
    } else {
        cout << "YES\n";
        for (auto [l,r] : ans) cout << l << " " << r << "\n"; 
    }
}

```

# K

容易发现，后根遍历的第一个点一定是最小的叶子 v，那么接下来我们只能返回到 v 的父亲 u，对于 u，我们有两种抉择，一种是认为 u 是我们所选定的根，那么我们会将 u 的除 v 以外的子树按照最小叶子排序依次 dfs，最终将 u 加入答案；或者认为 u 还有父亲，那么我们会将 u 的除 v 以外的子树按照最小叶子排序，不妨假设有 k 个子树，先依次 dfs 前 k−1 个，然后将 u 加入答案，然后 dfs 最后一个子树

至于实现，我们以最小的叶子为根建树，这样方便得到每个子树的最小叶子，同时在 dfs 的时候，我们记录现在是在向上还是向下

```cpp
const int N = 2e5 + 10;
int d[N],n,f[N];
vector <int> G[N];

void predfs(int x,int fa) {
    f[x] = 0x3f3f3f3f;
    bool cnt = 0;
    for (auto y : G[x]) {
        if (y == fa) continue;
        predfs(y,x);cnt = 1;
        f[x] = min(f[x],f[y]);
    }
    if (!cnt) f[x] = x;
}

bool cmp(int x,int y) {
    return f[x] < f[y];
}
vector <int> ans;

void dfs(int x,int fa,int typ) {
    vector <int> son;
    for (auto y : G[x]) {
        if (y == fa) continue;
        // dfs(y,x);
        son.push_back(y);
    }
    if (son.empty()) {
        ans.push_back(x);
        return ;
    }
    sort(son.begin(),son.end(),cmp);
    if (typ) {
        for (int i = 0;i + 1< son.size();++i) dfs(son[i],x,0);
        if (x > f[son.back()]) dfs(son.back(),x,0),ans.push_back(x);
        else ans.push_back(x),dfs(son.back(),x,1);
    } else {
        
        for (int i = 0;i < son.size();++i) dfs(son[i],x,0);
        ans.push_back(x);
    }
}

void solve() {
    cin >> n;
    for (int i = 1;i <= n;++i) G[i].clear(),d[i] = 0;
    for (int i = 1;i < n;++i) {
        int x,y;cin >> x >> y;
        G[x].push_back(y);G[y].push_back(x);
        ++d[x],++d[y];
    }
    int root = 0;
    for (int i = 1;i <= n;++i) {
        if (d[i] == 1) {
            root = i;
            break;
        }
    }
    predfs(root,0);
    dfs(root,0,1);
    for (auto x : ans) cout << x << " ";
    cout << "\n";
    ans.clear();
}

```

