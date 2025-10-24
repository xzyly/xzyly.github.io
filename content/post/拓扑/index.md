---
date: 2025-10-23
title: 拓扑
slug: 拓扑
image: OIP.jpg
categories:
    - topology
---

## Introduction

### 什么是拓扑顺序
想象你有一系列任务，其中一些任务必须在另一些任务之前完成。拓扑排序就是将这些任务排成一个线性序列，保证每个任务的所有“前置任务”都出现在它自己之前。它只适用于有向无环图（DAG）——即任务间的依赖关系不会形成一个循环，否则就永远找不到起点。

### 意义
它的核心价值在于理清依赖，保证顺序。在复杂的依赖网络中，它能提供一个清晰、可执行的行动路线图，确保整个过程不会因为前置条件不满足而卡住。

### 实践应用场景
任务调度：操作系统或分布式系统用它来决定多个进程或作业的执行顺序。

依赖关系解析：软件包管理器（如 apt, npm）在安装软件时，用它来解析并安装所有必需的依赖库。

课程先修计划：帮你规划大学选课顺序，确保在修读高级课程前，你已经完成了所有必修的先修课程。

构建系统：像 Make 或 Gradle 这样的工具，用它来编译源代码，确定哪些模块需要先被编译，哪些可以并行编译。

## Topology Sorting Basics
拓扑排序为一系列存在依赖关系的任务找到一个可行的执行序列，使得对于任何两个任务 A 和 B，如果 A 必须在 B 之前完成，那么在序列中 A 就出现在 B 之前。

它有一个黄金前提：这些任务和依赖必须构成一个 有向无环图（DAG）。

有向：依赖关系是单向的（穿袜子 → 穿鞋）。

无环：绝不能出现循环依赖。比如“穿鞋之前要穿袜子”和“穿袜子之前要穿鞋”同时存在，这就死锁了，永远无法开始。

## Kahn's算法

### 算法步骤
算法步骤
初始化：计算每个节点的入度（有多少边指向它）

找起点：将所有入度为0的节点加入队列

处理节点：

从队列中取出节点，加入结果序列

将该节点的所有邻居的入度减1

如果某个邻居的入度变为0，将其加入队列

检查结果：如果结果序列包含所有节点，成功；否则说明有环

### 算法实现

#### 题目
洛谷：https://www.luogu.com.cn/problem/P1807

在有向无环图(DAG)中求最长路，拓扑排序是最高效的方法。我们按拓扑顺序递推，确保处理每个节点时，所有可能影响它的前驱节点都已经被计算完毕，从而正确更新最长距离

#### 代码
```cpp
#include <bits/stdc++.h>
using namespace std;

int n, m;
const int inf = -0x3f3f3f3f;
const int N = 1505;

vector<pair<int, int>> e[N];
int dist[N];
int degree[N];

int main(){
	ios_base::sync_with_stdio(false);
	cin.tie(0);

	cin >> n >> m;
	fill(dist+1, dist+n+1, inf);
	for(int i = 1; i <= m; i++){
		int u,v,w;
		cin >> u >> v >> w;
		e[u].push_back({v, w});
		degree[v]++;
	}
	dist[1] = 0;
	queue<int> q;
	for(int i = 1; i <= n; i++){
		if(!degree[i]) q.push(i);
	}
	while(!q.empty()){
		int u = q.front(); q.pop();
		for(auto [v, w]: e[u]){
			if(dist[u] != inf){
				dist[v] = max(dist[v], dist[u]+w);
			}
			degree[v]--;
			if(!degree[v]) q.push(v);
		}
	}
	if(dist[n] == inf) cout << "-1";
	else cout << dist[n];
}
```

## 基于DFS的拓扑排序

### 核心思想
想象你在规划一天的任务：有些任务必须在其他任务之前完成。DFS拓扑排序就像是用递归的方式探索这些依赖关系，沿着每条依赖链深入到底，然后逆序记录结果。

### 算法流程
任选一个未访问节点开始DFS

递归访问所有后继节点

当某个节点没有未访问的后继时，将其加入结果集

最终反转结果顺序，就得到了拓扑排序

### 算法实现
我们还是用最长路来做题目，但是其实在最长路这个题目上dfs的优势不能很好表现，大家可以自己去探索到底什么时候用什么

#### 代码
```cpp
#include <bits/stdc++.h>
using namespace std;

int n, m;
const int inf = -0x3f3f3f3f;
const int N = 1505;

vector<pair<int, int>> e[N];
int dist[N];
bool visited[N];
vector<int> topo_order;

void dfs(int u) {
    visited[u] = true;
    for (auto [v, w] : e[u]) {
        if (!visited[v]) {
            dfs(v);
        }
    }
    topo_order.push_back(u);
}

int main() {
    ios_base::sync_with_stdio(false);
    cin.tie(0);

    cin >> n >> m;
    fill(dist + 1, dist + n + 1, inf);
    
    for (int i = 1; i <= m; i++) {
        int u, v, w;
        cin >> u >> v >> w;
        e[u].push_back({v, w});
    }
    
    // DFS拓扑排序
    fill(visited + 1, visited + n + 1, false);
    for (int i = 1; i <= n; i++) {
        if (!visited[i]) {
            dfs(i);
        }
    }
    reverse(topo_order.begin(), topo_order.end());
    
    // 动态规划求最长路
    dist[1] = 0;
    for (int u : topo_order) {
        if (dist[u] != inf) {
            for (auto [v, w] : e[u]) {
                if (dist[v] < dist[u] + w) {
                    dist[v] = dist[u] + w;
                }
            }
        }
    }
    
    if (dist[n] == inf) cout << "-1";
    else cout << dist[n];
    
    return 0;
}
```

## END（这part和算法无关了）
拓扑对我来说还是很有哲学意义的。我们面对很多事情的时候总是喜欢想清楚去做，等我们花费很多时间终于找到了那些事自己想做且的做的事情，结果却发现许多自己想做的事情的前提是一系列自己不想做的事情。比如拿到好成绩是大家想得到的，加练等等是不想要的，但是后者确实前者的前提。或许我们不能仅仅想自己想要的，而是基于我们想要的去做一个拓扑排序，最后一个个去完成他们。
