---
title: 题解 CF1594C Make Them Equal
tags:
  - 构造
  - 数学
  - 数论
  - CodeForces
  - 题解
categories:
  - 题解
abbrlink: 1281241727
date: 2021-10-23 22:12:27
---

## 题意

输入一个整数 $n$ ，一个字符 $c$ 和一个长度为 $n$ 的字符串 $s$。

选择若干个数 $x$，对于 $s$ 的第 $i$ 个字符 $s_i$，如果 $i$ 不能被 $x$ 整除，$s_i$ 就会被替换为 $c$。

求最少的操作次数和任意一种方案。

## 思路

本题的关键是，不论 $s$ 的内容如何，答案最多只为 $2$。因为选取了 $n$ 和 $n - 1$ 后肯定会将 $s$ 的所有字符全部变成 $c$，所以只需要考虑是否只需要一步就能解决问题即可，如果不能就直接输出 `2  ` 和 `n n - 1`。

对于只需要一步的情况，直接枚举即可，复杂度 $O(n \ln n)$。

## 代码

```cpp
#include<iostream>
using namespace std;

const int MAXN = 300000 + 5;

char c, s[MAXN];
int n, m;

int main() {
    int T;

    cin >> T;
    while(T--) {
        cin >> n >> c >> (s + 1);
        int cnt = 0;
        for(int i = 1; i <= n; i++) {
            if(s[i] != c) {
                cnt++;
            }
        }
        if(cnt == 0) {
            cout << 0 << endl;
            continue;
        }
        int ans = 0;
        bool p = false;
        for(int i = 1; i <= n; i++) {
            bool flag = true;
            for(int j = i; j <= n; j += i) {
                if(s[j] != c) {
                    flag = false;
                }
            }
            if(flag) {
                ans = i;
                p = true;
                break;
            }
        }
        if(p) {
            cout << 1 << endl;
            cout << ans << endl;
        } else {
            cout << 2 << endl;
            cout << n << " " << n - 1 << endl;
        }
    }

    return 0;
}
```