---
title: Codeforces Round 1048 (Div. 2)
date: 2025-09-09
slug: Codeforces Round 1048 (Div. 2)
image: OIP.jpg
categories:
    - competition
---

## C

### 题目链接
https://codeforces.com/contest/2139/problem/C

### 题目大意
一定数量的蛋糕（2^(k+1)），二人平分（分别叫a,b），现在有人想要特定数量的蛋糕，有两个操作，一个是a分一半给b，一个是b分一半给a，直到a拿到特定的x个，问要多少步，和具体方案

### 思路
刚开始两人是平分的，我们来假设测试分过几次后的状态，a有m个，b有2^(k+1)-m个，如果m小于总数的一半说明
上一个操作一定是a分一半，反之亦然。所以我们可以反向操作。目标a有的数量为x，如果小于总数的一半说明上一个
操作是1，否则是2。

### 代码
```cpp
#include <bits/stdc++.h>
using namespace std;

#define endl '\n'
#define pii pair<int,int>
#define int long long

const int mod = 1e9+7;
const int inf = 0x3f3f3f;

void solve(){
    int k,x;
    cin >> k >> x;
    int s = 1LL << (k+1);
    int h = 1LL << k;
    int cur = x;
    vector<int> a;
    while(cur != h){
        if(cur < h){
            a.push_back(1);
            cur = cur * 2;
        }else{
            a.push_back(2);
            cur = cur * 2 - s;
        }
    }
    reverse(a.begin(), a.end());
    cout << a.size() << endl;
    for(int i = 0; i < (int)a.size(); i++){
        cout << a[i];
        if(i + 1 != (int)a.size()) cout << " ";
    }
    cout << endl;
}

signed main(){
    ios_base::sync_with_stdio(false);
    cin.tie(0);

    int t = 1;
    cin >> t;
    while(t--){
        solve();
    }
    return 0;
}
```

### 总结
中题考查就是一个反向思考来简化问题

## D

### 题目链接
https://codeforces.com/contest/2139/problem/D

### 题目大意
有一个排列数组，我们有两个操作让他们变得非递减。操作一可以将第i位和i+1位换位置，操作二将
i和i+2换位置且只能最多用一次。我们每次询问会给出左右边界，要我们返回在这个范围内的元素是否
用操作二爷不会减少操作次数，即总次数和只用操作一是一样的。

### 思路
我们那三个元素来进行分析，a，b，c（索引递增）。我们可以列出他们的大小关系，这里就不做证明，各位很容易就可以得到就是当a>b>c时，操作二会减少操作数量。   
然后我们来分析abc是否一定要相邻，结论是不要。因为我们可以通过操作一让他们相邻。   
那么我们要判断的就是被询问的左右边界是否包含了这样的三个数。方法也很简单，我们对于每一个i去先向前寻找一个大于a[i]的位置,设他为l，再向后寻找一个小于a[i]的位置设他为r，并存储在r位置对应的最大的l的位置（为什么要最大呢，因为往i后面继续遍历的时候可能找到同样的r但是l更大，此时记录更大的l这样后面的左右边界就更能包裹住）。

### 代码
```cpp
#include <bits/stdc++.h>
using namespace std;

#define endl '\n'
#define pii pair<int,int>
#define int long long

const int mod = 1e9+7;
const int inf = 0x3f3f3f;

void solve(){
    int n,q;
    cin >> n >> q;
    vector<int> a(n+1, 0),mxl(n+1, 0), mxr(n+1, 0),L(n+1, 0), prev(n+1, 0);
    for(int i = 1; i <= n; i++) cin >> a[i];
    for(int i = 1; i <= n; i++){
        mxl[i] = i - 1;
        while(mxl[i] > 0 && a[mxl[i]] < a[i]){
            mxl[i] = mxl[mxl[i]];
        }
    }
    for(int i = n; i >= 1; i--){
        mxr[i] = i + 1;
        while(mxr[i] <= n && a[mxr[i]] > a[i]){
            mxr[i] = mxr[mxr[i]];
        }
    }
    for(int i = 2; i < n; i++){
        if(mxl[i] > 0 && mxr[i] <= n){
            L[mxr[i]] = max(L[mxr[i]], mxl[i]);
        }
    }
    for(int i = 1; i <= n; i++)
        L[i] = max(L[i], L[i-1]);
    for(int i = 0; i < q; i++){
        int l, r;
        cin >> l >> r;
        if(l <= L[r]){
            cout << "NO" << endl;
        }else{
            cout << "YES" << endl;
        }
    }
}

signed main(){
    ios_base::sync_with_stdio(false);
    cin.tie(0);

    int t = 1;
    cin >> t;
    while(t--){
        solve();
    }
    return 0;
}
```

### 总结
这道题开始确实想过用线段树，现在也不太明白这个思路是不是很偏，这个就是去寻找不符合的点然后看看边界有没有覆盖到他们，线段树先存好各个的状态再去查询，感觉思路差不多但是线段树比较多余对这道题来说。