---
title: 「SCOI2009」windy 数
tags:
  - 数位DP
  - 题解
categories:
  - 题解
abbrlink: 1092015460
date: 2021-01-02 09:13:28
---

## 题意简述

不含前导零且相邻两个数字之差至少为 $2$ 的正整数被称为 windy 数。windy 想知道，在 $a$ 和 $b$ 之间，包括 $a$ 和 $b$ ，总共有多少个 windy 数？

## 题目分析

数位DP模板题

定义状态 $dp_{i, j}$ 为长度为 $i$ 的windy数中，最高位为 $j$ 的数的数目。

状态转移方程也很容易得到，可以直接从前一位转移过来：

$$dp_{i, j} = \sum dp_{i - 1, k} (|j - k| \geq 2)$$

主要的难点是在计算 $1$ 到 $n$ 之间的 windy 数，这可以分为几个步骤处理。

第一步，求出所有长度小于等于 $len - 1$ （$len$ 为 $n$ 的长度）的，以 $1$ ~ $9$ 为最高位的 windy 数的总数。

第二步，求出所有长度为 $len$，最高位小于 $n$ 的最高位的 windy 数的总数。

第三步，求出所有长度为 $len$，最高位等于 $n$ 的最高位的 windy 数的总数。这一步是最难想的，需要从第二位开始，依次枚举每一位，并累加所有以与上一位的差小于 $2$ 的数为最高位的 windy 数的总数。（前几位之前已经补足。）

## 代码

```cpp
#include<bits/stdc++.h>
using namespace std;

const int MAXN = 10 + 5;

int f[MAXN][MAXN];
int a[MAXN];

int solve(int n) {
    int ans = 0, len = 0;

    memset(a, 0, sizeof(a));
    while(n != 0) {
        a[++len] = n % 10;
        n /= 10;
    }
    //求出所有长度小于等于 len - 1 的，以 1 ~ 9 为最高位的 windy 数的总数
    for(int i = 1; i <= len - 1; i++) {
        for(int j = 1; j <= 9; j++) {
            ans += f[i][j];
        }
    }
    //求出所有长度为 len，最高位小于 n 的最高位的 windy 数的总数
    for(int i = 1; i < a[len]; i++) {
        ans += f[len][i];
    }
    //求出所有长度为 len，最高位等于 n 的最高位的 windy 数的总数
    for(int i = len - 1; i >= 1; i--) {
        //依次枚举每一位上的值
        for(int j = 0; j < a[i]; j++) {
            if(abs(j - a[i + 1]) >= 2) {
                //符合要求，累加
                ans += f[i][j];
            }
        }
        if(abs(a[i + 1] - a[i]) < 2) {
            break; //已经不是 windy 数了
        }
    }
    return ans;
}

int main() {
    int l, r;

    
    scanf("%d%d", &l, &r);
    for(int i = 0; i <= 9; i++) {
        f[1][i] = 1;
    }
    
    for(int i = 2; i <= 10; i++) {
        for(int j = 0; j <= 9; j++) {
            for(int k = 0; k <= 9; k++) {
                if(abs(k - j) >= 2) {
                    f[i][j] += f[i - 1][k];
                }
            }
        }
    }
    
    printf("%d", solve(r + 1) - solve(l));

    return 0;
}
```