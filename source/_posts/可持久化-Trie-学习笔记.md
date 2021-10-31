---
title: 可持久化 Trie 学习笔记
tags:
  - Trie
  - 字符串
  - 可持久化
  - 异或
  - 数据结构
  - 学习笔记
categories:
  - 学习笔记
abbrlink: 2541031305
date: 2021-10-31 23:53:39
---

## 前言

> 看到异或便想到 Trie，便想到可持久化 Trie，OIer 的想象力唯在这一层面能如此跃进。——智子

（这篇文章的诞生原因主要其实是 @apple365 上星期说的某句话。）

## Trie

众所周知，Trie 树是一种用来处理字符串问题的数据结构。

### P2580 于是他错误的点名开始了

[洛谷 P2580 于是他错误的点名开始了](https://www.luogu.com.cn/problem/P2580)

输入若干个字符串，查询每个字符串是否在给定的字符串中，或者有没有重复查询。

暴力显然会超时，哈希能解决但是非常丑陋，而 Trie 树可以优美地解决这个问题。Trie 树和核心思想是尽可能重复利用相同的前缀，将所有的字符串从前往后作为一条从根节点到叶子的路径插入到树中。

![](https://oi-wiki.org//string/images/trie1.png)

（图片转载自 OI-Wiki，遵循 CC BY-SA 4.0 协议。）

例如这道题，直接建立一棵 Trie 树并在末尾标记是否有字符串结束和是否已经被查询过即可。

```cpp
#include<cstdio>
#include<cstring>
using namespace std;

const int MAXN = 500000 + 5;

int ch[MAXN][26], end[MAXN];
char s[MAXN];
int n, m, tot;

void insert(char * s) {
	int p = 0, l = strlen(s);

	for(int i = 0; i < l; i++) {
		int v = s[i] - 'a';

		if(ch[p][v] == 0) {
			ch[p][v] = ++tot;
		}
		p = ch[p][v];
	}
	end[p] = 1;
}

int find(char * s) {
	int p = 0, l = strlen(s);

	for(int i = 0; i < l; i++) {
		int v = s[i] - 'a';

		if(ch[p][v] == 0) {
			return 0;
		}
		p = ch[p][v];
	}
	if(end[p] == 1) {
		end[p] = 2;
		return 1;
	} else {
		return 2;
	}
}

int main() {
	scanf("%d", &n);
	for(int i = 1; i <= n; i++) {
		scanf("%s", s);
		insert(s);
	}
	scanf("%d", &m);
	for(int i = 1; i <= m; i++) {
		scanf("%s", s);
		int res = find(s);
		if(res == 0) {
			printf("WRONG\n");
		} else if(res == 1) {
			printf("OK\n");
		} else {
			printf("REPEAT\n");
		}
	}

	return 0;
}
```

## 01 Trie

众所周知，Trie 树是一种用来处理字符串问题的数据结构。

但在实际的比赛和练习中，Trie 树也常常被用来处理一类“最大异或”的问题，即要在一定的限定条件下求出某种最大的异或和。这种形式的 Trie 树通常被称为“01 Trie”。

### P4551 最长异或路径

下面是一道简单的例题：

[洛谷 P4551 最长异或路径](https://www.luogu.com.cn/problem/P4551)

> 给定一棵 $n$ 个点的带权树，结点下标从 $1$ 开始到 $n$。寻找树中找两个结点，求最长的异或路径。

> 异或路径指的是指两个结点之间唯一路径上的所有边权的异或。

首先定义 $sum_u$ 为从根节点到节点 $u$ 的路径所有边的异或和，$sum$ 可以用一遍 DFS 预处理出来。设 $dis(u, v)$ 为从 $u$ 到 $v$ 的路径上所有边的异或和，则有：

$$dis(u, v) = sum_u \oplus sum_{LCA(u, v)} \oplus sum_v \oplus sum_{LCA(u, v)} =sum_u \oplus sum_v $$

所以这道题相当于求 $sum_1$ ~ $sum_n$ 的 $n$ 个数中的两个数的异或和的最大值。

将每一个 $sum_i$ 看作一个长度为 $32$ 的 $01$ 字符串，并将其全部插入一颗 Trie 树。（遍历时从前往后，因为求异或和时高位显然比低位影响更大。）然后对于每一个 $sum_u$，进行一个贪心的策略，每次从根节点出发尝试访问与当前值的这一位相反的子节点。例如如果当前值为 $(010110)$，则每次尝试访问的子节点依次为 $(101001)$，如果该子节点不存在则只好访问当前值的这一位的字节点。

示例代码如下。

```cpp
#include<bits/stdc++.h> 
using namespace std;

struct Line {
	int v, w;
};

const int MAXN = 2000000 + 5;

vector<Line> G[MAXN];
int t[MAXN][2], sum[MAXN];
int n, tot;

void dfs(int u, int par) {
	for(int i = 0; i < G[u].size(); i++) {
		int v = G[u][i].v, w = G[u][i].w;
		
		if(v == par) {
			continue;
		}
		sum[v] = sum[u] ^ w;
		dfs(v, u);
	}
}

void insert(int x) {
	int p = 0;
	
	for(int i = (1 << 30); i != 0; i >>= 1) {
		bool c = x & i;
		if(t[p][c] == 0) {
			t[p][c] = ++tot;
		}
		p = t[p][c];
	}
}

int query(int x) {
	int p = 0, res = 0;
	
	for(int i = (1 << 30); i != 0; i >>= 1) {
		bool c = x & i;
		if(t[p][!c]) {
			res += i;
			p = t[p][!c];
		} else {
			p = t[p][c];
		}
	}
	
	return res;
}

int main() {
	scanf("%d", &n);
	for(int i = 1; i <= n - 1; i++) {
		int u, v, w;
		
		scanf("%d%d%d", &u, &v, &w);
		G[u].push_back({v, w});
		G[v].push_back({u, w});
	}
	dfs(1, 0);
	
	for(int i = 1; i <= n; i++) {
		insert(sum[i]);
	}
	
	int ans = 0;
	for(int i = 1; i <= n; i++) {
		ans = max(ans, query(sum[i]));
	}
	printf("%d\n", ans);
	
	return 0;
}
```

## 可持久化 Trie

可持久化数据结构的特点是能够维护数据结构的历史版本，但在 OI 中，大部分情况下“可持久化”的重点是能被用来处理序列乃至值域（如著名的主席树）上的问题，下面这道题就是对可持久化数据结构在 OI 中用途的一个经典阐述。

### P4735 最大异或和

[洛谷 P4735 最大异或和](https://www.luogu.com.cn/problem/P4735)

> 给定一个非负整数序列 $\{a\}$，初始长度为 $n$。

> 有 $m$ 个操作，有以下两种操作类型：

> 1. `A x`：添加操作，表示在序列末尾添加一个数 $x$，序列的长度 $n+1$。

> 2. `Q l r x`：询问操作，你需要找到一个位置 $p$，满足 $l \le p \le r$，使得：  $a[p] \oplus a[p+1] \oplus ... \oplus a[N] \oplus x$ 最大，输出最大是多少。

类似的，设 $sum_i$ 为前 $i$ 个数的前缀异或和，那么题中要求的 $a[p] \oplus a[p+1] \oplus ... \oplus a[N] \oplus x$ 就等于 $sum_{p - 1} \oplus sum_n \oplus x$。

所以题意即要求 $sum_p \oplus sum_n \oplus x$ 的最大值，其中 $l - 1 \le p \le r - 1$

假如没有 $l - 1 \le p \le r - 1$ 的要求，这道题的解法就和上一道题一样，直接在 01 Trie 中贪心地查找 $sum_n \oplus x$ 的每一位的相反子节点即可。

加上  $p \le r - 1$ 的要求后，我们发现这相当于要求在第 $r - 1$ 个数刚被插入后的 Trie 树上查询答案，即需要访问这颗 Trie 树的历史版本，使用可持久化 Trie 即可。 **这就是可持久化思想在 OI 中的主要意义：将序列和值域上的问题转为历史版本之间的关系的问题，减少问题的维度。**

至于  $l - 1 \le p \le r - 1$ 的要求则需要一点小技巧。记 $end_u$ 为节点 $u$ 是第几个字符串的末尾节点，$latest_u$ 为以节点 $u$ 为根的子树内的所有节点的 $end$ 的最大值。当我们在可持久化 Trie 的第 $r - 1$ 个历史版本上查询时，只访问 $latest$ 大于等于 $l - 1$ 的节点，忽略其他的节点，这样就能保证最后找到的串所在的范围满足要求。

代码如下：

```cpp
#include<cstdio>
#include<algorithm>
using namespace std;

const int MAXN = 600000 + 5;

int ch[MAXN * 24][2], latest[MAXN * 24];
int s[MAXN], rt[MAXN];
int n, m, tot;

void insert(int i, int k, int last, int now) {
	if(k < 0) {
		latest[now] = i;
		return;
	}
	int c = s[i] >> k & 1;
	if(last) {
		ch[now][c ^ 1] = ch[last][c ^ 1];
	}
	ch[now][c] = ++tot;
	insert(i, k - 1, ch[last][c], ch[now][c]);
	latest[now] = max(latest[ch[now][0]], latest[ch[now][1]]);
}

int ask(int now, int val, int k, int left) {
	if(k < 0) {
		return s[latest[now]] ^ val;
	}
	int c = val >> k & 1;
	if(latest[ch[now][c ^ 1]] >= left) {
		return ask(ch[now][c ^ 1], val, k - 1, left);
	} else {
		return ask(ch[now][c], val, k - 1, left);
	}
}

int main() {
	scanf("%d%d", &n, &m);
	latest[0] = -1;
	rt[0] = ++tot;
	insert(0, 23, 0, rt[0]);
	for(int i = 1; i <= n; i++) {
		int x;
		
		scanf("%d", &x);
		s[i] = s[i - 1] ^ x;
		rt[i] = ++tot;
		insert(i, 23, rt[i - 1], rt[i]);
	}
	for(int i = 1; i <= m; i++) {
		char op[2];
		
		scanf("%s", op);
		if(op[0] == 'A') {
			int x;
			
			scanf("%d", &x);
			rt[++n] = ++tot;
			s[n] = s[n - 1] ^ x;
			insert(n, 23, rt[n - 1], rt[n]);
		} else {
			int l, r, x;
			
			scanf("%d%d%d", &l, &r, &x);
			printf("%d\n", ask(rt[r - 1], x ^ s[n], 23, l - 1));
		}
	}
	
	return 0;
}
```
