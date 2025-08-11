---
title: The 2024 CCPC Shandong Invitational Contest and Provincial Collegiate Programming Contest
categories: 解题报告
tags: []
date: 2025-04-22 03:29:00
---
8/13 +748 新队伍第一次训练 感觉还不错。

和队友开出了 H 将成为本场高光！

<!--more-->

# A

签到题，简单二分，但是注意 `check()` 里是有可能爆 `long long` 的。

```cpp
#define li __int128
int n,k;
const int N = 1e3 + 10;
int t[N],l[N],w[N];
bool check(int x) {
    li ans = 0;
    for (int i = 1;i <= n;++i) {
        li inter = t[i] * l[i] + w[i];
        li rep = x / inter;
        rep = rep * l[i];
        li res = x % inter;
        ans += (li)min((li)l[i],res / t[i]);
        ans += rep;
    }
    return ans >= k;
}
void solve() {
    cin >> n >> k;
    for (int i = 1; i <= n; i++) {
        cin >> t[i] >> l[i] >> w[i];
    }
    int l = 0,r = 4e18;
    while (l + 1 < r) {
        int mid = (l + r) / 2;
        if (check(mid)) r = mid;
        else l = mid;
    }
    cout << r << "\n";
}
```

# C

我读完题给队友，队友直接秒了。不愧是 CF 大神（）

考虑这种线段覆盖问题，我们只需要考虑某个点上有几条线段，记当前线段个数为 $cnt_i$，我们只需要在每个七点统计一下，然后求 $\prod (n - cnt_i)$ 即可。太牛了队友。

```cpp

const int N = 5e5 + 10;
int n,k;
pii p[N];
void solve() {
    cin >> n >> k;
    for (int i = 1; i <= n; i++) {
        cin >> p[i].first >> p[i].second;
    }
    vector <pii> vec;
    for (int i = 1;i <= n;++i) {
        vec.emplace_back(p[i].first,1);
        vec.emplace_back(p[i].second+1,-1);
    }
    sort(vec.begin(),vec.end());
    int ans = 1;
    for (auto [p,typ] : vec) {
        if (typ == 1) {
            ans = ans * k % 998244353;
            k--;
        } else {
            ++k;
        }
    }
    cout << ans << "\n";
}
```

# D

队友开的，数学题。

考虑暴力模拟整个过程，不难发现迭代次数是 $O(\sqrt n)$ 水平的，暴力模拟即可。

```cpp
#include<bits/stdc++.h>
 
using namespace std;

#define ULL unsigned long long
#define int long long
#define ll __int128
#define endl '\n'
typedef pair<int, int> pii; 

void solve() {
    int p,a,b,q,c,d,m,t;
    cin>>p>>a>>b>>q>>c>>d>>m>>t;

    if(m<p){
        cout<<m<<endl;
        return;
    }

    while(1){
        int cnt=m/p;

        int time=(a+c)*cnt+b+d;
        int buy=p*cnt;
        int cha=(q-p)*cnt;

        int c=((cnt+1)*p-m-1+cha)/cha;

        int newc=t/time;
        if(c<=newc){
            m=m+(q-p)*cnt*c;
            t-=time*c;
        }else{
            c=newc;
            m=m+(q-p)*cnt*c;
            t-=time*c;
            break;
        }
    }

    t-=b+d;
    m+=max(0LL,(t)/(a+c)*(q-p));
    cout<<m<<endl;

}

signed main() {
    ios::sync_with_stdio(false);
    cin.tie(nullptr);
    int T = 1;
    cin >> T;
    while (T--) {
        solve();
    }
    return 0;
}
```

# F

显然只要每次贪心地取最大的后缀和累加即可，注意只用对后 $n-1$ 个后缀排序，因为 $k=1$ 时答案是唯一的

```cpp
#include<bits/stdc++.h>
 
using namespace std;

#define ULL unsigned long long
#define int long long
#define ll __int128
#define endl '\n'
typedef pair<int, int> pii; 

void solve() {
    int n;
    cin>>n;

    vector<int>a(n+1),sum(n+2);

    for(int i=1;i<=n;i++){
        cin>>a[i];
    }

    priority_queue<int>pq;

    for(int i=n;i>=1;i--){
        sum[i]=sum[i+1]+a[i];
        if(i!=1)pq.push(sum[i]);
        //cerr<<sum[i]<<" ";
    }

    cout<<sum[1]<<" ";
    int ans=sum[1];

    while(pq.size()){
        auto x=pq.top();pq.pop();
        ans+=x;
        cout<<ans<<" ";
    }

    cout<<endl;

}

signed main() {
    ios::sync_with_stdio(false);
    cin.tie(nullptr);
    int T = 1;
    cin >> T;
    while (T--) {
        solve();
    }
    return 0;
}

```

# H

过于 OI 味道的工业题了。

首先这个题第一步应该是想到怎么建图，根据我们想考虑的是交点，所以优先考虑是把行冲突点当成二分图左边点，列冲突点当成二分图右边点。考虑怎么刻画我们想要的答案？我们考虑每个交点，显然应该把对应的一组冲突连边。那么求出最小覆盖也就是最大匹配，就是我们想求的东西。

思路没有特别难，主要写的过程需要耐心，但是你怎么知道队友给了个建图，主播敲了200行一发过了😋这下本场高光了

```cpp
#include<bits/stdc++.h>
 
using namespace std;

#define ULL unsigned long long
#define int long long
#define ll __int128
#define endl '\n'
#define mp make_pair
typedef pair<int, int> pii; 
const int N = 1e3 + 10;
pii p[N];
vector <int> G[N];
int n,L,R,m;
bool cmp(pair<pii,int> a,pair<pii,int> b) {
    if (a.first.second == b.first.second) {
        if (a.first.first == b.first.first) {
            return a.second < b.second;
        } else {
            return a.first.first < b.first.first;
        }
    } else {
        return a.first.second < b.first.second;
    }
}

int match[N],ans;
bool vis[N];
bool dfs(int x) {
    for (auto y : G[x]) {
        if (!vis[y]) {
            vis[y] = 1;
            if (!match[y] || dfs(match[y])) {
                match[y] = x;
                return 1;
            }
        }
    }
    return 0;
}
int ln,rn;
int Hungary() {
    ans = 0;
    memset(match,0,sizeof match);
    for (int i = 1;i <= ln;++i) {
        memset(vis,0,sizeof vis);
        if (dfs(i)) ++ans;
    }
    // for (int i = 1;i <= rn;++i) cerr << match[ln + i] << " ";
    // cerr << endl;
    return ans;
}
void solve() {
    vector <pii> poiL,poiR;
    cin >> n;
    vector <pair<pii,int> > vec;
    for (int i = 1;i <= n;++i) {
        cin >> p[i].first >> p[i].second;
        vec.push_back(mp(p[i],i));
    }
    cin >> m;
    for (int i = 1;i <= m;++i) {
        int x,y;cin >> x >> y;
        vec.push_back(mp(mp(x,y),0));
    }
    sort(vec.begin(),vec.end());
    unordered_map <int,int> l;
    //先处理行冲突
    for (auto [p,typ] : vec) {
        int x = p.first,y = p.second;
        // 
        if (typ == 0) {
            l[x] = 0;
        } else {//cerr << x << " " << y << " " << l[x] << " " << typ  << "\n";
            if (l[x] > 0) {
                poiL.push_back(mp(l[x],typ));
            }
            l[x] = typ;
        }
    }
    // for (auto [x,y] : poiL) {
    //     cerr << "x:" << x << " y:" << y << "\n";
    //     cerr << p[x].first << " " << p[x].second << "\n";
    //     cerr << p[y].first << " " << p[y].second << "\n";
    // }
    // cerr << "----\n";
    unordered_map <int,int> r;
    for (auto [p,typ] : vec) {
        int x = p.first,y = p.second;
        if (typ == 0) {
            r[y] = 0;
        } else {//cerr << x << " " << y << " " << r[y] << " " << typ  << "\n";
            if (r[y] > 0) {
                poiR.push_back(mp(r[y],typ));
            }
            r[y] = typ;
        }
    }
    // for (auto [x,y] : poiR) {
    //     cerr << "x:" << x << " y:" << y << "\n";
    //     cerr << p[x].first << " " << p[x].second << "\n";
    //     cerr << p[y].first << " " << p[y].second << "\n";
    // }
    ln = poiL.size(),rn = poiR.size();
    for (int i = 0;i <= ln + rn;++i) {
        G[i].clear();
    } 
    for (int i = 0;i < ln;++i) {
        int lpos = poiL[i].first,rpos = poiL[i].second;
        pii pL = p[lpos],pR = p[rpos];
        int l = pL.second,r = pR.second;
        int P1 = pL.first;
        for (int j = 0;j < rn;++j) {
            int qwqwq = poiR[j].first,tat = poiR[j].second;
            // if (poiL[i] == lpos || poiL[i] == rpos || qwqwq == lpos || qwqwq == rpos) continue;
            pii pU = p[qwqwq],pD = p[tat];
            int x = pU.first,y = pD.first;
            int P2 = pU.second;
            if ((P2 < r && P2 > l) && (P1 > x && P1 < y)) {
                // cerr << l << ' ' << r << " "<< P2 << "\n";
                // cerr << x << ' ' << y << " "<< P1 << "\n";
                G[i+1].push_back(j + ln+1);
                G[j + ln+1].push_back(i+1);
                // cerr << i << " " << j + ln << "\n";
            }
        }
    }
    Hungary();
    vector <pii> tmpans;
    for (int i = 0;i < rn;++i) {
        if (match[ln + i+1]) {
            // cerr << "ln+i!" << ln + i << "\n";
            // for (int i = 1;i <= rn;++i) cerr << match[ln + i] << " ";
            int lpos = poiL[match[ln + i+1]-1].first,rpos = poiR[i].first;
            // cerr << lpos << " " << rpos << "\n";
            tmpans.push_back(mp(p[lpos].first,p[rpos].second));
            // cerr << p[lpos].first << " " << p[rpos].second << "\n";
            match[match[ln+i+1]] = i + 1;
        }
    }

    bool invalid = 0;
    for (int i = 0;i < ln;++i) {
        if (!match[i + 1]) {
            int lpos = poiL[i].first,rpos = poiL[i].second;
            pii pL = p[lpos],pR = p[rpos];
            int l = pL.second,r = pR.second;
            if (l + 1 == r) {
                invalid = 1;
            }
            tmpans.push_back(mp(pL.first,pL.second + 1));
        }
    }
    for (int j = 0;j < rn;++j) {
        if (!match[j+ln + 1]) {
            int qwqwq = poiR[j].first,tat = poiR[j].second;
            // if (poiL[i] == lpos || poiL[i] == rpos || qwqwq == lpos || qwqwq == rpos) continue;
            pii pU = p[qwqwq],pD = p[tat];
            int x = pU.first,y = pD.first;
            int P2 = pU.second;
            if (x + 1 == y) {
                invalid = 1;
            }
            tmpans.push_back(mp(x + 1,P2));
        }
    }
    if (invalid) {
        cout << -1 << "\n";
        return ;
    }
    cout << tmpans.size() << "\n";
    for (auto [x,y] : tmpans) cout << x << " " << y << "\n";
}

signed main() {
    ios::sync_with_stdio(false);
    cin.tie(nullptr);
    int T = 1;
    cin >> T;
    while (T--) {
        solve();
    }
    return 0;
}
```

# I

签到，不多说。

```cpp
#include<bits/stdc++.h>
 
using namespace std;

#define ULL unsigned long long
#define int long long
#define ll __int128
#define endl '\n'
typedef pair<int, int> pii; 

void solve() {
    string s;cin >> s;
    bool flag = 0;
    int n = s.size();
    for (int i = 0;i < n;++i) {
        if (s[i] == s[(i + n-1) % n]) {
            cout << i << "\n";
            return ;
        }
    }
    cout << "-1\n";
}

signed main() {
    ios::sync_with_stdio(false);
    cin.tie(nullptr);
    int T = 1;
    cin >> T;
    while (T--) {
        solve();
    }
    return 0;
}


```

# J

思维难度甚至比 H 高点。

考虑依然是进行 Kruskal 的过程，但是实际上有用的边只需要考虑链接两集合的和集合内部链接的边，在 Kruskal 的过程中分类讨论：

1. 如果两边是连接两个集合

   - 集合内部两个都不连通：把两个都搞联通，代价是 $(a_x + a_y - 1)\times w$
   - 一个联通一个不连通：把不联通的搞到联通的一个点上，代价为 $a_x\times w$
   - 两个都联通：只链接一条边即可，$w$

2. 如果是内部集合

   直接连边即可，根据从小到大考虑性质，肯定是比情况 1.1 优秀的。

```cpp
#include<bits/stdc++.h>
 
using namespace std;

#define ULL unsigned long long
#define int long long
#define ll __int128
#define endl '\n'
#define mp make_pair
typedef pair<int, int> pii; 
const int MOD=998244353;
const int N = 1e3 + 10;
int a[N],b[N][N],n,typ[N],c[N];
int f[N];

int find(int x){
    if (f[x] != x){
        int root = find(f[x]);
        typ[x] |= typ[f[x]];
        f[x] = root;
    }
    return f[x];
}
int solve1() {
    int ans = 0;
    vector <pair<int,pii>  > ed;
    for (int i = 1;i <= n;++i) f[i] = i,typ[i] = 0;
    for (int i = 1;i <= n;++i) {
        for (int j = 1;j <= i;++j) {
            ed.push_back(mp(b[i][j],mp(i,j)));
        }
    }
    sort(ed.begin(),ed.end());
    int cnt = 0;
    for (auto &[w,e] : ed) {
        int x = e.first,y = e.second;
        x = find(x),y = find(y);
        if (x != y) {
            if (!typ[x] && !typ[y]) {
                int val = (a[x] + a[y] - 1) * w;
                ans += val;
                f[x] = y;
                typ[x] = typ[y] = 1;
            } else if (typ[x] && !typ[y]){
                int val = a[y] * w;
                f[y] = x;
                typ[y] = 1;
                ans += val;
            } else if (!typ[x] && typ[y]){
                int val = a[x] * w;
                f[x] = y;
                typ[x] = 1;
                ans += val;
            } else {
                f[x] = y;
                ans += w;
            }
            ++cnt;
        } else {
            if (typ[x]) continue;
            else {
                typ[x] = 1;
                ans += (a[x] - 1) * w;
            }
        }
        // if (cnt == n -1 ) break;
    }
    // cerr << "ans1:" << ans << endl;
    return ans;
}
void solve() {
    cin >> n;
    for (int i = 1;i <= n;++i) {
        cin >> a[i];typ[i] = 0;
    }
    for (int i = 1;i <= n;++i) {
        for (int j = 1;j <= n;++j) {
            cin >> b[i][j];
        }
        c[i] = b[i][i];
    }
    int ans1 = solve1();
    // cerr << ans1 << " " << ans2 << "\n";
    // cerr << ans << "\n";
    cout << ans1 << "\n";
}

signed main() {
    ios::sync_with_stdio(false);
    // cin.tie(nullptr);
    int T = 1;
    cin >> T;
    while (T--) {
        solve();
    }
    return 0;
}
```

