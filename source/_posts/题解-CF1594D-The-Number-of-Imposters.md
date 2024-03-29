---
title: 题解 CF1594D The Number of Imposters
tags:
  - 并查集
  - CodeForces
  - 题解
categories:
  - 题解
abbrlink: 413439375
date: 2021-10-23 22:09:02
---

## 题意

有 $n$ 个玩家和 $m$ 句话，每个玩家可能是船员或内鬼，每句话的形式为“玩家 $i$ 说玩家 $j$ 是船员/内鬼”，求最多可能有多少个内鬼。（如果自相矛盾，输出 $-1$。）

PS：题目背景出自国外的著名游戏  _Among Us_ ，中文名为《我们之中》，也被叫做《内鬼杀》或《太空狼人杀》。游戏背景设置于太空，玩家将扮演船员（英语：Crewmate）或内鬼（英语：Impostor）之一。船员的目标是找出伪装者并完成任务，而伪装者的目标是杀死所有船员而不被发现。

## 思路

这种题目很容易让人联想到并查集。就像 [NOI2001 食物链](https://www.luogu.com.cn/problem/P2024) 一样，这道题可以用“扩展域”并查集来解决。

用 $x$ 表示玩家 $x$ 是船员，$x + n$ 表示船员 $x$ 是内鬼。

对于一句话，如果是“玩家 $x$ 说玩家 $y$ 是船员”，就意味着：

- 如果这句话为真，那么玩家 $x$ 和玩家 $y$ 都是船员。

- 如果这句话为假，那么玩家 $x$ 和玩家 $y$ 都是内鬼。

同理，如果是“玩家 $x$ 说玩家 $y$ 是内鬼”，就意味着：

- 如果这句话为真，那么玩家 $x$ 是船员，玩家 $y$ 是内鬼。

- 如果这句话为假，那么玩家 $x$ 是内鬼，玩家 $y$ 是船员。

按照这个规则合并并查集，然后检查如果存在 $find(x) = find(x + n)$，说明无解，输出 `-1`。

否则，对于每一个点 $i$，考虑分别包含 $i$ 和 $i + n$ 的两个集合，哪个集合包含的内鬼多就选取哪个集合，并累加答案。

## 代码

```cpp
#include<iostream>
#include<algorithm>
#include<cstring>
using namespace std;

const int MAXN = 200000 + 5;

int f[MAXN * 2], vis[MAXN * 2], cnt[MAXN * 2];
int n, m, p;

int find(int k) {
    return k == f[k] ? k : f[k] = find(f[k]);
}

int main() {
    int T;

    cin >> T;
    while(T--) {
        char s[20];

        memset(vis, 0, sizeof(vis));
        memset(cnt, 0, sizeof(cnt));
        memset(f, 0, sizeof(f));
        cin >> n >> m;
        for(int i = 1; i <= n * 2; i++) {
            f[i] = i;
        }
        for(int i = 1; i <= m; i++) {
            int x, y;

            cin >> x >> y >> s;
            if(s[0] == 'c') {
                f[find(x)] = f[find(y)];
                f[find(x + n)] = find(y + n);
            } else {
                f[find(x)] = find(y + n);
                f[find(x + n)] = find(y);
            }
        }
        bool flag = true;
        for(int i = 1; i <= n; i++) {
            if(find(i) == find(i + n)) {
                flag = false; // 无解
            }
        }
        if(!flag) {
            cout << -1 << endl;
            continue;
        }
        p = 0;
        for(int i = 1; i <= n * 2; i++) {
            if(i > n) {
                cnt[find(i)]++; //统计每个集合的内鬼总数
            }
        }
        int ans = 0;
        for(int i = 1; i <= n; i++) {
            if(vis[find(i)] || vis[find(i + n)]) {
                continue; //如果这个点已经访问过，就不再访问
            }
            ans += max(cnt[find(i)], cnt[find(i + n)]); //选取内鬼较多的集合，加上其内鬼总数
            vis[find(i)] = true;
            vis[find(i + n)] = true; //标记访问
        }
        cout << ans << endl;
    }

    return 0;
}
```