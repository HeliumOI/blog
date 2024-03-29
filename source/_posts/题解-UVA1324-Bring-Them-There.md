---
title: 题解 UVA1324 Bring Them There
categories: 题解
tags:
  - UVa
  - 题解
  - 最大流
  - 分层图
abbrlink: 2022708006
date: 2022-01-28 19:03:18
---

## 题目分析

可以看出这道题应该用网络流解决，但如果单纯使用网络流的话，“每条隧道需要一天时间来通过”就无法处理了。

在这道题中，我们注意到每台计算机能否移动与图当前的状态有关，而图当前的状态又与“时间”这一参数有关，所以我们建图时就应该考虑时间的影响，换句话说，建分层图。假设一共运送了 $T$ 天，那么我们就可以将每个点 $u$ 拆成 $T + 1$ 个点 $u_0$、$u_2$、$u_3$、……、$u_T$，$u_i$ 表示第 $i$ 天的节点 $u$，具体实现时则使用 $u_i = u + in$ 的方式来表示。

对于一条无向边 $(u, v)$，我们从 $u_i$ 向 $v_{i + 1}$ 连容量为 $1$ 的边，从 $v_i$ 向 $u_{i + 1}$ 连容量为 $1$ 的边，表示飞船可以花一天的时间从 $u$ 到 $v$ 或从 $v$ 到 $u$。同时，对于每一个节点 $u$，我们都从 $u_i$ 向 $u_{i + 1}$ 连容量为正无穷的边，表示飞船可以待在原地不动。

首先能想到二分 $T$ 。但也可以一层一层建图（每次将 $T$ 增加 $1$。），每建完一层就在上一次的残量网络上跑从 $s_0$ 到 $t_T$ 的最大流，直到最大流达到 $k$ 为止。（达到 $k$ 时立即停止，这一点可以通过在增广时对初始流量进行限制来实现。）

另一个比较棘手的问题是如何输出方案。事实上，第 $i$ 天时所有计算机的动向的信息就保存在第 $i$ 层分层图中，如果从 $u_i$ 到 $v_{i + 1}$ 的边满流，就意味着在第 $i + 1$ 天有一台计算机从 $u$ 移动到 $v$。（因为所有计算机彼此之间没有区别，所以从 $u_i$ 到 $v_{i + 1}$ 的边和从 $v_i$ 到 $u_{i + 1}$ 的边都满流可以被视作相互抵消。）

所以我们可以先记录所有“移动事件”，再对于每一个从 $(u, v)$ 的移动，随便找一台当前位置为 $u$ 的计算机，将它的位置改为 $v$ 并输出。当然，因为一台计算机一天只能移动一次，我们必须记录每台计算机当天是否已经移动过了，如果已经移动过了就不再移动。

## 代码实现

```cpp
#include<bits/stdc++.h>
using namespace std;

struct Edge {
    int u, v;
};

const int MAXN = 5000 + 5;
const int MAXM = 200000 + 5;
const int MAXK = 50 + 5;
const int INF = INT_MAX;

Edge e[MAXM];
pair<int, int> mov[MAXM];
queue<int> q;
int pos[MAXK];
bool blk[MAXK];
int head[MAXN], to[MAXM], nxt[MAXM], flow[MAXM], cap[MAXM];
int tot = 1;
int dep[MAXN], cur[MAXN];
int n, m, k, s, T, t;
int cnt, ans;

void init() {
    memset(head, 0, sizeof(head));
    tot = 1;
}

void addEdge(int u, int v, int c, int f) {
    tot++;
    to[tot] = v;
    nxt[tot] = head[u];
    flow[tot] = f;
    cap[tot] = c;
    head[u] = tot;
}

bool bfs() {
    memset(dep, 0, sizeof(dep));
    while(!q.empty()) q.pop();
    q.push(s);
    dep[s] = 1;
    while(!q.empty()) {
        int u = q.front(); q.pop();
        for(int i = head[u]; i != 0; i = nxt[i]) {
            if(cap[i] > flow[i] && dep[to[i]] == 0) {
                dep[to[i]] = dep[u] + 1;
                q.push(to[i]);
                if(to[i] == t) return true;
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
    for(int& i = cur[u]; i != 0; i = nxt[i]) {
        if(cap[i] > flow[i] && dep[to[i]] == dep[u] + 1) {
            int k = dinic(to[i], min(rest, cap[i] - flow[i]));
            if(k == 0) {
                dep[to[i]] = 0;
            }
            flow[i] += k;
            flow[i ^ 1] -= k;
            rest -= k;
            if(rest == 0) break;
        }
    }
    return limit - rest;
}

int maxflow(int limit) {
    int res = 0;
    while(bfs()) {
        memcpy(cur, head, sizeof(head));
        res += dinic(s, limit - res); //流量限制，确保流量正好为 $k$
        if(res == limit) {
            break;
        }
    }
    return res;
}

void output() {
    for(int i = 1; i <= k; i++) {
        pos[i] = s;
    }
    int now = 2;
    for(int i = 1; i <= cnt; i++) {
        memset(blk, 0, sizeof(blk));
        ans = 0;
        for(int j = 1; j <= m; j++) {
            if(flow[now] == 1 && flow[now + 2] == 0) {
                mov[++ans] = make_pair(e[j].u, e[j].v);
            }
            if(flow[now] == 0 && flow[now + 2] == 1) {
                mov[++ans] = make_pair(e[j].v, e[j].u);
            }
            now += 4;
        }
        now += n * 2;
        printf("%d", ans);
        for(int j = 1; j <= ans; j++) {
            for(int p = 1; p <= k; p++) {
                if(blk[p]) {
                    continue;
                }
                if(pos[p] == mov[j].first) {
                    pos[p] = mov[j].second;
                    printf(" %d %d", p, pos[p]);
                    blk[p] = true;
                    break;
                }
            }
        }
        printf("\n");
    }
}

int main() {
    while(scanf("%d%d%d%d%d", &n, &m, &k, &s, &T) == 5) {
        init();
        for(int i = 1; i <= m; i++) {
            scanf("%d%d", &e[i].u, &e[i].v);
        }
        cnt = 0;
        int sum = 0;
        while(sum < k) {
            cnt++;
            t = cnt * n + T;
            for(int i = 1; i <= m; i++) {
                addEdge(e[i].u + (cnt - 1) * n, e[i].v + cnt * n, 1, 0);
                addEdge(e[i].v + cnt * n, e[i].u + (cnt - 1) * n, 0, 0);
                addEdge(e[i].v + (cnt - 1) * n, e[i].u + cnt * n, 1, 0);
                addEdge(e[i].u + cnt * n, e[i].v + (cnt - 1) * n, 0, 0);
            }
            for(int i = 1; i <= n; i++) {
                addEdge(i + (cnt - 1) * n, i + cnt * n, INF, 0);
                addEdge(i + cnt * n, i + (cnt - 1) * n, 0, 0);
            }
            sum += maxflow(k - sum);
        }
        printf("%d\n", cnt);
        output();
    }

    return 0;
}
```

