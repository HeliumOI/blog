---
title: UVA11383 Golden Tiger Claw 题解
categories: 题解
tags:
  - UVa
  - 题解
  - KM
  - 二分图带权匹配
abbrlink: 2111054984
date: 2022-04-09 15:04:22
---

## 分析

我们会发现这道题本质上是一个线性规划问题：

$$h_i + l_j \ge w_{i,j}$$
$$h_i \ge 0 (\forall 1 \le i \le n)$$
$$l_j \ge 0 (\forall 1 \le j \le n)$$
$$\operatorname{minimize} \sum_{i = 1}^{n} h_i + \sum_{j = 1}^{n} l_j$$

我们考虑它的对偶线性规划。

### 对偶线性规划

所谓对偶线性规划是指：

- 原问题中的每个变量都变为对偶问题中的一个限制条件；
- 原问题中的每个限制条件都变为对偶问题中的一个变量；
- 原问题若是求目标函数的最大值，则对偶问题是求最小值，反之亦然。

另外还有“强对偶定理”，通俗的来说就是原问题的最优解等于对偶问题的最优解。

例如本题的对偶线性规划为：

$$\sum_{j = 1}^{n} y_{i, j} \le 1 (\forall 1 \le i \le n) $$
$$\sum_{i = 1}^{n} y_{i, j} \le 1 (\forall 1 \le j \le n) $$
$$y_{i, j} \ge 0 $$
$$\operatorname{maximize} \sum_{(i, j)} w_{i, j} y_{i, j}$$

注意到这正好是二分图最大权匹配问题，所以可以用 KM 算法解决。

（绕了一个大圈……事实上 KM 算法本身的原理就是解决二分图最大权匹配问题的对偶问题“二分图最小顶标和问题”，即本问题。换句话说，KM 算法本身就会保证算法结束后顶标最小。）

## 代码

KM 算法的模版。

```cpp
#include<bits/stdc++.h>
using namespace std;

const int MAXN = 500 + 5;
const int INF = 1e9;

int e[MAXN][MAXN], la[MAXN], lb[MAXN], slack[MAXN];
int match[MAXN], last[MAXN];
bool va[MAXN], vb[MAXN];
int n;

bool dfs(int u, int fa) {
    va[u] = true;
    for(int v = 1; v <= n; v++) {
        if(vb[v]) continue;
        if(la[u] + lb[v] == e[u][v]) {
            vb[v] = true;
            last[v] = fa;
            if(!match[v] || dfs(match[v], v)) {
                match[v] = u;
                return true;
            }
        } else if(slack[v] > la[u] + lb[v] - e[u][v]) {
            slack[v] = la[u] + lb[v] - e[u][v];
            last[v] = fa;
        }
    }
    return false;
}

void KM() {
    fill(match + 1, match + n + 1, 0);
    fill(last + 1, last + n + 1, 0);
    for(int i = 1; i <= n; i++) {
        la[i] = -INF;
        lb[i] = 0;
        for(int j = 1; j <= n; j++) {
            la[i] = max(la[i], e[i][j]);
        }
    }
    for(int i = 1; i <= n; i++) {
        fill(va + 1, va + n + 1, false);
        fill(vb + 1, vb + n + 1, false);
        fill(slack + 1, slack + n + 1, INF);
        int st = 0; match[st] = i;
        while(match[st]) {
            if(dfs(match[st], st)) break;
            int delta = INF;
            for(int j = 1; j <= n; j++) {
                if(!vb[j] && delta > slack[j]) {
                    delta = slack[j];
                    st = j;
                }
            }
            for(int j = 1; j <= n; j++) {
                if(va[j]) la[j] -= delta;
                if(vb[j]) lb[j] += delta; else slack[j] -= delta;
            }
            vb[st] = true;
        }
        while(st) {
            match[st] = match[last[st]];
            st = last[st];
        }
    }
}

int main() {
    while(scanf("%d", &n) != EOF) {
        for(int i = 1; i <= n; i++) {
            for(int j = 1; j <= n; j++) {
                scanf("%d", &e[i][j]);
            }
        }
        KM();
        int ans = 0;
        for(int i = 1; i <= n; i++) {
            printf("%d ", la[i]);
            ans += la[i];
        }
        printf("\n");
        for(int i = 1; i <= n; i++) {
            printf("%d ", lb[i]);
            ans += lb[i];
        }
        printf("\n");
        printf("%d\n", ans);
    }

    return 0;
}
```
