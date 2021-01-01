---
title: 题解「CF161D」Distance in Tree
tags:
  - 树
  - 树形DP
categories:
  - 题解
abbrlink: 1350111558
date: 2021-01-01 20:01:39
---

## 题意简述

给定一颗$N$个节点的树，求树上长度为$K$的路径的数量

### 输入

第一行两个数字$N$，$K$，如题意

接下来的$N−1$行中，每行两个整数$u,v$表示一条树边$(u,v)$

### 输出

一个整数$ans$，如题意

### 数据范围

$1 \leq N \leq 50000$

$1 \leq K \leq 500$

## 题目分析

虽然此题可以看出是个点分治模板，但由于$K$的范围较小（小于等于$500$），所以树形DP也可以解决这道题

定义状态$f_{u,i}$为从$u$开始的路径中长度为$i$的路径的数量，可以很容易得出状态转移方程

$$f_{u, i} = \sum_{v \in son_u} f_{v, i - 1}$$

回到题目本身，题目中要求我们找出所有长度为$K$的路径

以$u$为根节点考虑，可以将所有的路径分为两种：经过点$u$的和不经过点$u$的

对于后者，可以直接递归处理；对于前者，我们可以以点$u$将其分为两条路径，两条路径的长度和为$K$。所以，根据乘法原理，我们只需累加`f[v][i] * f[u][k - i - 1]`即可。

另外，累加答案必须在更新$f_{u, i}$之前，因为在更新之前，$f_{u, i}$是从点$u$开始，不经过点$v$的长度为$i$的路径数量，乘以$f_{u, K - i - 1}$就可以得到经过$u, v$的长度为$K$的路径数量。但如果先更新$f_{u, i}$，则$f_{u, i}$是从点$u$开始，经过点$v$和其他子节点的长度为$i$的路径数量，会出现重复：即一条路径从$v$走到$u$，又走回$v$的“重复”现象

## 代码

```cpp
#include<bits/stdc++.h>
using namespace std;

const int MAXN = 50000 + 5;
const int MAXK = 500 + 5;

vector<int> G[MAXN];
int f[MAXN][MAXK];
int n, k;
long long ans;

void dp(int u, int par) {
	f[u][0] = 1;
	for(int i = 0; i < G[u].size(); i++) {
		int v = G[u][i];
		
		if(v == par) {
			continue;
		}
		dp(v, u);
        //这两重循环的顺序不能反，因为如果ans累加前就更新了f[u][i]，累加时就会出现重复，即两条路径的边重合
		for(int i = 0; i <= k - 1; i++) {
			ans += (long long)f[v][i] * f[u][k - i - 1]; //乘法原理
		}
		for(int i = 0; i <= k - 1; i++) {
			f[u][i + 1] += f[v][i]; //更新f[u][i]
		}
	}
}

int main() {
	scanf("%d%d", &n, &k);
	for(int i = 1; i <= n - 1; i++) {
		int u, v;
		
		scanf("%d%d", &u, &v);
		G[u].push_back(v);
		G[v].push_back(u);
	}
	dp(1, 0);
	printf("%lld\n", ans);
	
	return 0;
}
```