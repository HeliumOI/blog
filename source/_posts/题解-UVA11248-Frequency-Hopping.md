---
title: 题解 UVA11248 Frequency Hopping
categories: 题解
tags:
  - UVa
  - 题解
  - 最大流
abbrlink: 2012849851
date: 2022-01-28 13:39:06
---

## 题目分析

对于第一种情况，可以直接跑一遍最大流，如果最大流大于或等于 $C$ 就可以直接输出 `possible`。

如果最大流小于 $C$，那么肯定是满流的边将其“卡住了”，所以应该依次枚举每条满流的边（没有满流的边增加容量也一定没用），将这些边的容量一个一个改为 $C$，再对于每条边修改后都跑一次最大流，如果这条边的容量改为 $C$ 后最大流大于或等于 $C$ 就将其记录到答案中。

除此之外还有两个个必要的优化：一、没必要每次都从头开始跑一遍最大流。在跑完第一遍之后将残量网络备份下来，每次改变边的容量时在残量网络上跑最大流，少做很多无用功；二，不需要每次都把最大流求出来，可以在最大流达到 $C$ 时停止增广。

## 代码实现

```cpp
#include<bits/stdc++.h>
using namespace std;

struct Edge {
    int from, to, cap, flow, nxt;
};

const int MAXN = 100 + 5;
const int MAXM = 10000 * 2 + 1;
const int INF = INT_MAX;

vector<pair<int, int> > ans;
queue<int> q;
Edge e[MAXM], tmp[MAXM];
int head[MAXN];
int tot = 1;
int dep[MAXN], cur[MAXN];
int n, m, k, s, t;

void init() {
    memset(head, 0, sizeof(head));
    tot = 1;
    memset(e, 0, sizeof(e));
}

void addEdge(int u, int v, int c, int f) {
    tot++;
    e[tot].from = u;
    e[tot].to = v;
    e[tot].cap = c;
    e[tot].flow = f;
    e[tot].nxt = head[u];
    head[u] = tot;
}

bool bfs() {
    memset(dep, 0, sizeof(dep));
    while(!q.empty()) q.pop();
    q.push(s);
    dep[s] = 1;
    while(!q.empty()) {
        int u = q.front(); q.pop();
        for(int i = head[u]; i != 0; i = e[i].nxt) {
            if(e[i].cap > e[i].flow && dep[e[i].to] == 0) {
                dep[e[i].to] = dep[u] + 1;
                q.push(e[i].to);
                if(e[i].to == t) return true;
            }
        }
    }
    return false;
}

int dinic(int u, int limit) {
    if(u == t || limit == 0) {
        return limit;
    }
    int rest = limit;
    for(int& i = cur[u]; i != 0; i = e[i].nxt) {
        if(e[i].cap > e[i].flow && dep[e[i].to] == dep[u] + 1) {
            int k = dinic(e[i].to, min(rest, e[i].cap - e[i].flow));
            if(k == 0) {
                dep[e[i].to] = 0;
            }
            e[i].flow += k;
            e[i ^ 1].flow -= k;
            rest -= k;
            if(rest == 0) break;
        }
    }
    return limit - rest;
}

int maxflow() {
    int res = 0;
    while(bfs()) {
        memcpy(cur, head, sizeof(head));
        res += dinic(s, INF);
        if(res >= k) {
            break;
        }
    }
    return res;
}

int main() {
    int kase = 0;
    while(1) {
        kase++;
        scanf("%d%d%d", &n, &m, &k);
        if(n == 0 && m == 0 && k == 0) {
            break;
        }
        init();
        for(int i = 1; i <= m; i++) {
            int u, v, c;

            scanf("%d%d%d", &u, &v, &c);
            addEdge(u, v, c, 0);
            addEdge(v, u, 0, 0);
        }
        s = 1; t = n;
        int sum = maxflow();
        if(sum >= k) {
            printf("Case %d: possible\n", kase);
            continue;
        }
        ans.clear();
        memcpy(tmp, e, sizeof(e)); //备份残量网络
        for(int i = 2; i <= tot; i += 2) {
            if(e[i].cap == e[i].flow) {
                e[i].cap = k;
                if(sum + maxflow() >= k) {
                    ans.push_back(make_pair(e[i].from, e[i].to));
                }
                memcpy(e, tmp, sizeof(tmp)); //跑完最大流后将残量网络还原
            }
        }
        if(ans.size() == 0) {
            printf("Case %d: not possible\n", kase);
            continue;
        }
        sort(ans.begin(), ans.end());
        printf("Case %d: possible option:(%d,%d)", kase, ans[0].first, ans[0].second);
        for(int i = 1; i < ans.size(); i++) {
            printf(",(%d,%d)", ans[i].first, ans[i].second);
        }
        printf("\n");
    }

    return 0;
}
```

