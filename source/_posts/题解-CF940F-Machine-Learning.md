---
title: 题解 CF940F Machine Learning
categories: 题解
tags:
  - CodeForces
  - 题解
  - 莫队
abbrlink: 4081384378
date: 2022-01-15 12:15:26
---

## 思路

一道带修莫队模板题。

看到题意中“单点修改，区间查询数字出现次数 $mex$” 的操作后，不难想到用带修莫队维护，但时间复杂度的正确性需要证明。

假设区间当前区间的 $mex$ 为 $n$，则这个区间的数的个数至少为 

$$\sum\limits_{i = 1}^{n}i = \dfrac{n(n + 1)}{2}$$

所以任何区间的答案肯定在 $O(\sqrt{n})$ 级别，可以直接暴力查找 $mex$。

代码中也有一些要注意的细节：$x_i$ 最大可以为 $10^9$，所以序列中的数和修改中的数都必须离散化。

统计时开两个数组，一个数组 `cnt` 统计每个数出现了几次，`tot` 统计这个出现次数出现了几次。由于 `add` 和 `del` 操作都会使 `tot` 数组改变两次，所以每次操作都要更新两遍 $mex$。具体操作如下：

```cpp
void add(int x) {
    tot[cnt[x]]--;
    if(tot[cnt[x]] == 0) {
        mex = min(mex, cnt[x]); //可能出现了更小的答案
    }
    cnt[x]++;
    tot[cnt[x]]++;
    if(tot[cnt[x]] == 1 && mex == cnt[x]) { //当前答案已经出现，向上寻找符合要求的 mex
        for(int i = mex; i <= siz; i++) {
            if(tot[i + 1] == 0) {
                mex = i + 1;
                break;
            }
        }
    }
}

void del(int x) {
    tot[cnt[x]]--;
    if(tot[cnt[x]] == 0) {
        mex = min(mex, cnt[x]); //可能出现了更小的答案
    }
    cnt[x]--;
    tot[cnt[x]]++;
    if(tot[cnt[x]] == 1 && mex == cnt[x]) { //当前答案已经出现，向上寻找符合要求的 mex
        for(int i = mex; i <= siz; i++) {
            if(tot[i + 1] == 0) {
                mex = i + 1;
                break;
            }
        }
    }
}
```

## 代码

```cpp
#include<bits/stdc++.h>
using namespace std;

struct Query {
    int l, r, t, id;
};

struct Change {
    int pos, v;
};

const int MAXN = 100000 * 2 + 5;

Query q[MAXN];
Change c[MAXN];
int v[MAXN], cnt[MAXN], tot[MAXN], ans[MAXN], a[MAXN];
int cntQ, cntC;
int n, m, t, siz, mex = 1;

bool cmp(Query& a, Query& b) {
    if(a.l / t != b.l / t) {
        return a.l < b.l;
    }
    if(a.r / t != b.r / t) {
        return a.r < b.r;
    }
    return a.t < b.t;
}

void add(int x) {
    tot[cnt[x]]--;
    if(tot[cnt[x]] == 0) {
        mex = min(mex, cnt[x]);
    }
    cnt[x]++;
    tot[cnt[x]]++;
    if(tot[cnt[x]] == 1 && mex == cnt[x]) {
        for(int i = mex; i <= siz; i++) {
            if(tot[i + 1] == 0) {
                mex = i + 1;
                break;
            }
        }
    }
}

void del(int x) {
    tot[cnt[x]]--;
    if(tot[cnt[x]] == 0) {
        mex = min(mex, cnt[x]);
    }
    cnt[x]--;
    tot[cnt[x]]++;
    if(tot[cnt[x]] == 1 && mex == cnt[x]) {
        for(int i = mex; i <= siz; i++) {
            if(tot[i + 1] == 0) {
                mex = i + 1;
                break;
            }
        }
    }
}

void modify(int i, int t) {
    if(q[i].l <= c[t].pos && c[t].pos <= q[i].r) {
        del(a[c[t].pos]);
        add(c[t].v);
    }
    swap(a[c[t].pos], c[t].v);
}

int main() {
    scanf("%d%d", &n, &m);
    t = pow(n, 2.0 / 3);
    for(int i = 1; i <= n; i++) {
        scanf("%d", &a[i]);
        v[++siz] = a[i];
    }
    for(int i = 1; i <= m; i++) {
        int op;

        scanf("%d", &op);
        if(op == 1) {
            cntQ++;
            scanf("%d%d", &q[cntQ].l, &q[cntQ].r);
            q[cntQ].t = cntC;
            q[cntQ].id = cntQ;
        } else {
            cntC++;
            scanf("%d%d", &c[cntC].pos, &c[cntC].v);
            v[++siz] = c[cntC].v;
        }
    }
    sort(v + 1, v + siz + 1);
    siz = unique(v + 1, v + siz + 1) - (v + 1);
    for(int i = 1; i <= n; i++) {
        a[i] = lower_bound(v + 1, v + siz + 1, a[i]) - v;
    }
    for(int i = 1; i <= cntC; i++) {
        c[i].v = lower_bound(v + 1, v + siz + 1, c[i].v) - v;
    }
    sort(q + 1, q + cntQ + 1, cmp);
    int l = 1, r = 0, t = 0;
    for(int i = 1; i <= cntQ; i++) {
        while(l > q[i].l) add(a[--l]);
        while(r < q[i].r) add(a[++r]);
        while(l < q[i].l) del(a[l++]);
        while(r > q[i].r) del(a[r--]);
        while(t < q[i].t) modify(i, ++t);
        while(t > q[i].t) modify(i, t--);
        ans[q[i].id] = mex;
    }
    for(int i = 1; i <= cntQ; i++) {
        printf("%d\n", ans[i]);
    }

    return 0;
}

```
