---
title: Codeforces Round 756 (Div. 3) 解题报告
tags:
  - 构造
  - 贪心
  - 二分
  - ST表
  - CodeForces
  - 题解
categories:
  - 题解
abbrlink: 1747842812
date: 2021-11-29 23:42:26
---

# A - Make Even 

## 题意

输入一个数 $n$，每次操作可以翻转 $n$ 的前 $l$ 位，问最少要进行多少次操作才能使 $n$ 变成偶数。

## 思路

分情况讨论：

1. 如果这个数是偶数，答案显然为 0。
2. 如果这个数是奇数，第一位为偶数，只需翻转整个数即可，答案为 1。
3. 如果这个数是奇数，其中第 $l$ 位是偶数，那么先翻转 $1$ ~ $l$ 位将它翻转到第一位，再翻转整个数即可。
4. 否则，即如果这个数所有数位上都是奇数，那么不管进行多少次操作都无法变成偶数，输出 -1。

## 代码

```cpp
#include<cstdio>
#include<cstring>
using namespace std;

int main() {
    int T;

    scanf("%d", &T);
    while(T--) {
        int n, a[15], l = 0;

        scanf("%d", &n);
        int t = n;
        bool flag = false;
        while(t != 0) {
            a[++l] = t % 10;
            t /= 10;
            if(a[l] % 2 == 0) {
                flag = true;
            }
        }
        if(n % 2 == 0) {
            printf("%d\n", 0);
            continue;
        }
        if(a[l] % 2 == 0) {
            printf("%d\n", 1);
            continue;
        }
        if(flag) {
            printf("%d\n", 2);
            continue;
        }
        printf("%d\n", -1);
    }

    return 0;
}

```

# B - Team Composition: Programmers and Mathematicians 

## 题意

$a$ 个程序员和 $b$ 个数学家组队，每队 4 人。每支队伍至少要有一名程序员和一名数学家。求最多能组多少支队伍。

## 思路

容易想到答案为 $\lfloor \frac{a + b}{4} \rfloor$，但如果 $a$ 和 $b$ 中较小的数小于较大的数的三分之一，就无法得到这个答案。此时答案应该是 $min\{a, b\}$，所以需要先特判一下。

## 代码

```cpp
#include<cstdio>
#include<algorithm>
using namespace std;

int main() {
    int T;

    scanf("%d", &T);
    while(T--) {
        int a, b;

        scanf("%d%d", &a, &b);
        if(a > b) {
            swap(a, b);
        }
        if((long long)a * 3 <= b) {
            printf("%d\n", a);
        } else {
            printf("%d\n", (a + b) / 4);
        }
    }

    return 0;
}

```

# C - Polycarp Recovers the Permutation 

## 题意

有一个元素个数为 $n$ 的数组 $p$，我们每次执行以下操作：

取 $p$ 最左边和最右边的数**中较小的一个**，如果选的是最左边，就加入新数组 $a$ 的左边，否则加入 $a$ 的右边。

现在给出 $a$，求 $p$。如果无解，输出 `-1`；如果多组解，输出任意一组即可。

（[关于原翻译的错误。](https://www.luogu.com.cn/discuss/385807)）

## 思路

~~说实话第一眼看到这道题时首先想到了今年 CSP-S T3。~~

这道题其实是一个模拟，我们只需要按照题意从最终结果反推初始状态就行。

题目中每次取左右较小值，我们反过来，取 $a$ 最左边和最右边的数中较**大**的一个，如果选的是最左边，就加入原数组 $p$ 的左边，否则加入 $p$ 的右边。

判断无解也比较简单：如果有解，$n$ 要么在 $a$ 的最左边，要么在最右边。

## 代码

```cpp
#include<cstdio>
#include<deque>
using namespace std;

const int MAXN = 200000 + 5;

int a[MAXN], ans[MAXN];
int n;

int main() {
    int T;

    scanf("%d", &T);
    while(T--) {
        deque<int> q;
        scanf("%d", &n);
        for(int i = 1; i <= n; i++) {
            scanf("%d", &a[i]);
        }
        if(a[1] != n && a[n] != n) {
            printf("%d\n", -1);
            continue;
        }
        int s = 1, t = n;
        while(s <= t) {
            if(a[s] > a[t]) {
                q.push_front(a[s++]);
            } else {
                q.push_back(a[t--]);
            }
        }
        for(int i = 0; i < n; i++) {
            printf("%d ", q[i]);
        }
        printf("\n");
    }

    return 0;
}

```

# D - Weights Assignment For Tree Edges 

## 题意

输入一棵 $n$ 个节点的有根树和一个排列 $p$，你需要给树上的每条边赋予一个权值，使得每个 $p_{i - 1}$ 到根结点的距离小于 $p_i$ 到根结点的距离。

## 思路

因为题目要求每个 $p_{i - 1}$ 到根结点的距离小于 $p_i$ 到根结点的距离，不妨令 $p_i$ 到根结点的距离为 $i - 1$，设为 $dis_{p_i}$，那么 $w_i$ 就等于 $dis_{p_i} - dis_{p_{fa_i}}$。

然后就需要判断无解。如果我们需要计算 $p_i$ 的答案时，$p_i$ 的父节点的答案还没被算出来，则可以直接判断为无解。另外，如果第一个 $p_1$ 不是根结点，显然无解，因为距离根结点最近的节点显然就是根结点本身，不可能是其他节点。

## 代码

```cpp
#include<cstdio>
#include<cstring>
#include<vector>
using namespace std;

const int MAXN = 200000 + 5;

vector<int> G[MAXN];
int fa[MAXN], pos[MAXN], ans[MAXN], dis[MAXN];
int n, rt;

int main() {
    int T;

    scanf("%d", &T);
    while(T--) {
        scanf("%d", &n);
        for(int i = 1; i <= n; i++) {
            scanf("%d", &fa[i]);
            G[i].clear();
        }
        rt = 0;
        for(int i = 1; i <= n; i++) {
            if(fa[i] == i) {
                rt = i;
                continue;
            }
            G[fa[i]].push_back(i);
        }
        for(int i = 1; i <= n; i++) {
            scanf("%d", &pos[i]);
        }
        int flag = 1;
        memset(ans, -1, sizeof(ans));
        memset(dis, 0, sizeof(dis));
        ans[rt] = 0;
        for(int i = 1; i <= n; i++) {
            if(ans[fa[pos[i]]] == -1) {
                flag = 0;
                break;
            }
            if(pos[i] == rt) {
                continue;
            }
            dis[pos[i]] = dis[pos[i - 1]] + 1;
            ans[pos[i]] = dis[pos[i]] - dis[fa[pos[i]]];
        }
        if(!flag || pos[1] != rt) {
            printf("-1\n");
            continue;
        }
        for(int i = 1; i <= n; i++) {
            printf("%d ", ans[i]);
        }
        printf("\n");
    }

    return 0;
}

```

# E1 - Escape The Maze (easy version) 

## 题意

Vlad 和他的朋友们玩游戏。游戏场地是一棵 $n$ 个节点的有根树。Vlad 开始在 1 号节点，他的朋友们在另外 $k$ 个节点。Vlad 每移动一次， Vlad 的朋友们可以任意沿着边移动一次也可以不移动。Vlad 如果能走到一个叶子结点，并且途中没有碰到任何一个朋友就获胜了。问 Vlad 是否有办法做到必胜。

## 思路

跑两遍 BFS，一边计算 Vlad 到每个节点的最短距离，一遍计算 Vlad 的朋友们到每个节点的最短距离。最后，如果存在一个叶子结点，Vlad 到它的距离比 Vlad 的朋友们到它的距离更短，Vlad 只需要移动到这个叶子结点就能获胜。

（一下为了方便说明，记 Vlad 到每个节点的距离为 $tmp_i$，Vlad 的朋友们到每个节点的最短距离为 $dis_i$。满足条件的叶节点 $i$ 即 $tmp_i < dis_i$。）

这个方法的正确性很容易证明：从根结点到每个叶子节点只有一条路径，如果这条路径上有任意一个节点$j$ 出现了 $tmp_j \ge dis_j$ 的情况，这个叶子结点 $i$ 肯定也有 $tmp_i \ge dis_i$。换句话说，只要叶子结点满足 $tmp_i < dis_i$，整条路径上所有节点也都满足 $tmp_i < dis_i$，即 Vlad 离该节点的距离比所有朋友都更近。

所以对于满足 $tmp_i < dis_i$ 的叶子节点，Vlad 只要一开始就选择走这条路径，就可以在任何一个朋友到达该路径上的任何一个节点前到达该节点，并最终获胜。

## 代码

```cpp
#include<cstdio>
#include<vector>
#include<queue>
using namespace std;

const int MAXN = 200000 + 5;

vector<int> G[MAXN];
bool vis[MAXN];
int pos[MAXN], dis[MAXN], tmp[MAXN];
int n, k;

void bfs1() {
    queue<int> q;

    for(int i = 1; i <= n; i++) {
        dis[i] = 0;
        vis[i] = false;
    }
    q.push(1);
    vis[1] = true;
    while(!q.empty()) {
        int u = q.front(); q.pop();

        for(int i = 0; i < G[u].size(); i++) {
            int v = G[u][i];

            if(vis[v]) {
                continue;
            }
            dis[v] = dis[u] + 1;
            vis[v] = true;
            q.push(v);
        }
    }
    for(int i = 1; i <= n; i++) {
        tmp[i] = dis[i];
    }
}

void bfs2() {
    queue<int> q;

    for(int i = 1; i <= n; i++) {
        dis[i] = 0;
        vis[i] = false;
    }
    for(int i = 1; i <= k; i++) {
        q.push(pos[i]);
        vis[pos[i]] = true;
    }
    while(!q.empty()) {
        int u = q.front(); q.pop();

        for(int i = 0; i < G[u].size(); i++) {
            int v = G[u][i];

            if(vis[v]) {
                continue;
            }
            dis[v] = dis[u] + 1;
            vis[v] = true;
            q.push(v);
        }
    }
}

int main() {
    int T;

    scanf("%d", &T);
    while(T--) {
        scanf("%d%d", &n, &k);
        for(int i = 1; i <= n; i++) {
            G[i].clear();
        }
        for(int i = 1; i <= k; i++) {
            scanf("%d", &pos[i]);
        }
        for(int i = 1; i <= n - 1; i++) {
            int u, v;

            scanf("%d%d", &u, &v);
            G[u].push_back(v);
            G[v].push_back(u);
        }
        bfs1();
        bfs2();
        bool flag = false;
        for(int i = 2; i <= n; i++) {
            if(G[i].size() == 1 && tmp[i] < dis[i]) {
                flag = true;
                break;
            }
        }
        if(flag) {
            printf("YES\n");
        } else {
            printf("NO\n");
        }
    }

    return 0;
}

```

# F - ATM and Students 

## 题意

[题面翻译](https://www.luogu.com.cn/paste/wrrwu53t)

给定一个长度为 $n$ 的序列 $a$ 和一个整数 $s$，求一段最长的区间 $[l,r]$ 使得 $\forall i \in [l,r] , s + \sum\limits_{j = l}^i a_j \geq 0$。

## 思路

对于这种“求最长的符合要求的区间的问题”，由于它满足单调性，所以我们可以用“枚举左端点，二分右端点”的常用套路求解。

我们先求出 $a$ 数组的前缀和 $sum$，那么问题就变成了求一段最长的区间 $[l,r]$ 使得 $s + \min_{j = l}^r sum_j - sum_{i - 1} \geq 0$。中间求区间最小值的部分可以用 ST 表预处理出来，然后 $O(1)$ 查询；而这个式子本身则可以用二分答案求解。

总复杂度 $O(n \ log\ n)$。

## 代码

```cpp
#include<cstdio>
#include<algorithm>
using namespace std;

const int MAXN = 200000 + 5;
const int MAXB = 21;

long long st[MAXN][MAXB], lg[MAXN];
long long a[MAXN];
int n, s;

void init() {
    for(int i = 1; i <= n; i++) {
		st[i][0] = a[i];
		if(i >= 2) {
			lg[i] = lg[i / 2] + 1;
		}
	}
	for(int j = 1; j < MAXB; j++) {
		for(int i = 1; i + (1 << j) - 1 <= n; i++) {
			st[i][j] = min(st[i][j - 1], st[i + (1 << (j - 1))][j - 1]);
		}
	}
}

long long ask(int l, int r) {
	int t = lg[r - l + 1];
	return min(st[l][t], st[r - (1 << t) + 1][t]);
}

int main() {
    int T;

    scanf("%d", &T);
    while(T--) {
        scanf("%d%d", &n, &s);
        for(int i = 1; i <= n; i++) {
            scanf("%lld", &a[i]);
            a[i] += a[i - 1];
        }
        init();
        int ansl, ansr, ans = 0;
        for(int i = 1; i <= n; i++) {
            int l = i, r = n, res = 0;
            while(l <= r) {
                int mid = (l + r) / 2;

                if(s + ask(i, mid) >= a[i - 1]) {
                    res = mid;
                    l = mid + 1;
                } else {
                    r = mid - 1;
                }
            }
            if(res == 0) {
                continue;
            }
            if(res - i + 1 > ans) {
                ans = res - i + 1;
                ansl = i;
                ansr = res;
            }
        }
        if(ans == 0) {
            printf("%d\n", -1);
        } else {
            printf("%d %d\n", ansl, ansr);
        }
    }

    return 0;
}

```