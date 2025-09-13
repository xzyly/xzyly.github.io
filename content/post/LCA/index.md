---
date: 2025-09-13
title: LCA(最近公共祖先)
slug: LCA(最近公共祖先)
image: OIP.jpg
categories:
    -LCA
---

这里主要分享怎么用倍增法和Tarjan来解决LCA， 这两个方法也是在工作生活中应用基本最广的。    
    
这篇博客经历许多挫折，开始在写状压的题，开始直接用bfs爆搜，t了8个点，后面用LCA，用倍增法实现
,t了5个点，我也为是算法问题，去搜了下tarjan，学了好半天还是没有搞懂，后面发现倍增法可以过，只要小小优化
一下就好了。后面ac后还是不甘心没看懂tarjan，又去看了好些博客和问了AI，得出来最基本的理解，
所以想要深入了解的，慎入。

## 倍增法

### 核心思想
假设我们要找u和v的最近公共祖先，我们先判断u和v的深度，我们先让他们跳到同一个深度，再一起跳，直到他们公共祖先的下一层    
    
为什么叫倍增呢，因为我们跳的步幅是2^j, 比如s[i][j]就是从i开始，向上跳2^j，这样我们就可以更快到达目的地。    
    
这里要注意几个地方，就是比如我们要更新s[i][j]的状态时，我们先看s[i][j-1]，如果s[i][j-1]已经超出界限，那s[i][j]肯定也超过了，那么s[i][j]无法更新，则s[i][j] = s[i][j-1]。而如果未超呢，那么s[i][j] = s[s[i][j-1]][j-1],即从i先跳2^(j-1)，再跳2^(j-1)。

### 例子
n个点，q次询问，问两个点的最近公共祖先是谁

### 代码
```cpp
#include <bits/stdc++.h>
using namespace std;

#define endl '\n'
#define pii pair<int,int>
#define int long long

const int N = 1e5+5;
const int LOG = 20; // enough since 2^20 > 1e5

int n, q, root;
vector<int> G[N];
int up[N][LOG]; // up[v][j] = 2^j-th ancestor of v
int depth[N];

// DFS to initialize parent and depth
void dfs(int u, int p) {
    up[u][0] = p;
    for (int i = 1; i < LOG; i++) {
        if (up[u][i-1] != -1)
            up[u][i] = up[up[u][i-1]][i-1];
        else
            up[u][i] = -1;
    }
    for (int v : G[u]) {
        if (v == p) continue;
        depth[v] = depth[u] + 1;
        dfs(v, u);
    }
}

// Lift node u up by k steps
int lift(int u, int k) {
    for (int i = 0; i < LOG; i++) {
        if (k & (1 << i)) {
            u = up[u][i];
            if (u == -1) break;
        }
    }
    return u;
}

// Find LCA using binary lifting
int lca(int u, int v) {
    if (depth[u] < depth[v]) swap(u, v);
    // lift u up to the same depth as v
    u = lift(u, depth[u] - depth[v]);
    if (u == v) return u;

    for (int i = LOG - 1; i >= 0; i--) {
        if (up[u][i] != up[v][i]) {
            u = up[u][i];
            v = up[v][i];
        }
    }
    return up[u][0];
}

void solve() {
    cin >> n >> q >> root;
    for (int i = 1; i <= n; i++) {
        G[i].clear();
        depth[i] = 0;
        for (int j = 0; j < LOG; j++) up[i][j] = -1;
    }

    for (int i = 1; i < n; i++) {
        int u, v;
        cin >> u >> v;
        G[u].push_back(v);
        G[v].push_back(u);
    }

    depth[root] = 0;
    dfs(root, -1);

    for (int i = 0; i < q; i++) {
        int u, v;
        cin >> u >> v;
        cout << "LCA of query " << i << " is " << lca(u, v) << endl;
    }
}

signed main() {
    ios_base::sync_with_stdio(false);
    cin.tie(0);

    int t = 1;
    // cin >> t;
    while (t--) {
        solve();
    }
    return 0;
}
```

## Tarjan

### 核心思想
想象一下，你是一个侦探，正在调查一宗案件（求解LCA）。这棵家谱树就是你的案发现场。你的调查方式是一种特殊的深度优先搜索（DFS）：你从老祖宗（根节点）开始，沿着家谱一支一支地调查，彻底查完一支家族（子树），才会返回上一级做汇报。

在这个调查过程中，你有两个关键工具：

你的笔记本（并查集）：记录每个家族分支目前是归谁管的。

你的待办清单（查询列表）：记录你要查明的血缘关系，比如“小明和小红是什么关系？”。

分步情景演绎
我们还是用那棵树来做例子：

- 1（老祖宗）
  - 2
    - 4
    - 5
  - 3
查询：LCA(4, 5) 和 LCA(4, 3)

第一步：调查节点 1（老祖宗）
你（侦探）的行动：你站在了老祖宗 1 的位置。你在他的名字上画个圈（标记为灰色1），表示“我正在调查他”。

你的待办清单：你看了一眼，发现没有关于 1 的直接查询。

下一步：你决定按顺序调查，先调查他的大儿子 2 这一支。你暂时离开了 1，但你知道你最终会回到这里。

第二步：调查节点 2
你的行动：你来到了 2 的家。你画个圈（标记2）。

待办清单：没有关于 2 的查询。

下一步：你继续深入，先去调查 2 的大儿子 4。

第三步：调查节点 4
你的行动：你来到了 4 的家。你画个圈（标记4）。

检查待办清单：你发现有一个查询是“4 和 5 的关系”。你看了看 5，5 你还没去调查过（状态是白色0），你没有任何线索，所以这个查询暂时无法解决。

下一步：4 没有后代了，你对他家的调查完毕。你在 4 的名字上画个勾（标记为黑色2），这表示“这家我查完了，没问题”。

关键操作：现在你要回去向你的上级 2 汇报。汇报的时候，你说：“老大，你儿子 4 这一支我查完了，以后他们家的事就直接归你管了！” 这个“归你管”的操作，就是并查集的 unite(4, 2)。现在，如果你问“4 的负责人是谁？”，答案不再是 4 自己，而是 2。

第四步：回到节点 2，调查节点 5
你的行动：你回到了 2 这里。根据汇报，你把 4 的管理权收归己有。现在你开始调查 2 的二儿子 5。

你来到 5 的家，画个圈（标记5）。

检查待办清单：你发现有一个查询是“5 和 4 的关系”。你一看，4 的状态是已画勾（黑色2），说明 4 家已经被查完了。

关键破案：你现在想知道 4 是归谁管的？你查了一下你的笔记本（调用 find(4)），笔记本告诉你：4 现在归 2 管！太好了，那么这个 2 就是同时管着 4 和 5 的人，他就是他俩的最近公共上级（LCA）！ 你立刻记录：LCA(4,5) = 2。

下一步：5 没有后代了，调查完毕。你在 5 上画个勾（标记黑色2），然后回去向 2 汇报。汇报内容：“老二 5 这一支也查完了，也归你管了！” (unite(5, 2))。

第五步：回到节点 2，最终汇报
你的行动：你回到了 2 这里。2 的所有儿子都调查完毕了，所以你自己也在 2 上画个勾（标记黑色2）。

下一步：你回到你的上级，老祖宗 1 那里做汇报。你说：“老祖宗，您儿子 2 这一整支我都查完了，以后他们全家都归您直接管了！” (unite(2, 1))。现在，如果你问“2 的负责人是谁？”，答案会是 1；问“4 的负责人是谁？”，会先找到 2 的负责人是 1，所以答案也是 1（并查集的路径压缩效应）。

第六步：回到节点 1，调查节点 3
你的行动：你回到 1 这里。你开始调查他的二儿子 3 这一支。

你来到 3 的家，画个圈（标记3）。

检查待办清单：你发现有一个查询是“3 和 4 的关系”。你一看，4 的状态是已画勾（黑色2）。

关键破案：你马上查笔记本 4 归谁管（find(4)）。笔记本显示：4 所在的那一整支 (2 的家) 都已经归老祖宗 1 管了，所以 4 的负责人是 1！那么这个 1 就是同时管着 3 和 4 的人，他是他俩的最近公共祖先！ 你记录：LCA(4,3) = 1。

下一步：3 调查完毕，画勾，然后向 1 汇报 (unite(3, 1))。

第七步：全部调查完毕
你在 1 处画勾，整个调查结束。所有查询都已破案。

### 代码
```cpp
#include <bits/stdc++.h>
using namespace std;

#define endl '\n'
#define pii pair<int,int>
#define int long long

const int mod = 1e9+7;
const int inf = 0x3f3f3f;

template<typename Ty> inline void read(Ty &x) {
    short c = getchar(), f = 1;
    for (; c < '0' || c > '9'; c = getchar()) if (c == '-') f = -1;
    for (x = 0; c >= '0' && c <= '9'; c = getchar()) x = (x << 1) + (x << 3) + (c ^ 48);
    x *= f;
}

int wr_l = 0, wr_s[1000005];
template<typename Ty> inline void write(Ty x) {
    if (x == 0) return (void)(putchar('0'));
    if (x < 0) putchar('-'), x = -x;
    for (; x; x /= 10) wr_s[++wr_l] = x % 10 + '0';
    for (; wr_l; wr_l--) putchar(wr_s[wr_l]);
}

// n points, q times query
int n,q,root;
// assume the max vertex is N
const int N = 1e5+5;
// store the tree
vector<int> G[N];
// store the query
vector<pii> query[N];
// store the father
int parent[N];
// store the visited path
bool vis[N];
// store the ans
int ans[N];

int find(int x){
    if(x != parent[x]){
        parent[x] = find(parent[x]);
    }
    return parent[x];
}

void tarjan(int u){
    vis[u] = true;
    parent[u] = u;
    for(int v: G[u]){
        if(!vis[v]){
            tarjan(v);
            parent[v] = u;
        }
    }
    for(auto &q: query[u]){
        int v = q.first;
        int id = q.second;
        if(vis[v]){
            ans[id] = find(v);
        }
    }
}

void init(){
    for (int i = 1; i <= n; i++){
        G[i].clear();
        query[i].clear();
        vis[i] = false;
    }
}

void solve(){
    cin >> n >> q >> root;
    init();
    for (int i = 1; i < n; i++){
        int u, v;
        cin >> u >> v;
        G[u].push_back(v);
        G[v].push_back(u);
    }
    for (int i = 0; i < q; i++) {
        int u, v;
        cin >> u >> v;
        query[u].push_back({v, i});
        query[v].push_back({u, i});
    }
    tarjan(root);
    for (int i = 0; i < q; i++){
        cout << "LCA of query " << i << " is " << ans[i] << endl;
    }
}

signed main(){
    ios_base::sync_with_stdio(false);
    cin.tie(0);

    int t = 1;
    // cin >> t;
    while(t--){
        solve();
    }
    return 0;
}
```