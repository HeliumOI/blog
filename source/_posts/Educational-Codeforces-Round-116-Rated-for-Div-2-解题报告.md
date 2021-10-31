---
title: Educational Codeforces Round 116 (Rated for Div. 2) 解题报告
tags:
  - 构造
  - 数学
  - 数论
  - 贪心
  - 动态规划
  - DP
  - 组合数学
  - CodeForces
  - 题解
categories:
  - 题解
abbrlink: 485584732
date: 2021-11-01 01:07:00
---

# A - AB Balance

## 题意

给出一个只由字符 $a$ 和 $b$ 组成的字符串 $s$，求最少的修改使 $s$ 中 $ab$ 的出现次数和 $ba$ 的出现次数相等。

## 思路

$s$ 中 $ab$ 和 $ba$ 都只会在连续段之间出现，并且交替出现，所以出现次数最多相差$1$。如果一共有奇数段，两者的出现次数肯定相等；如果有偶数段，只需要修改第一个字符就会变成奇数段。

所以只要出现次数不相等就修改第一个字符即可。

## 代码

```cpp
#include<cstdio>
#include<cstring>
using namespace std;

const int MAXN = 100 + 5;

char s[MAXN];
int n;

int main() {
    int T;

    scanf("%d", &T);
    while(T--) {
        scanf("%s", s + 1);
        n = strlen(s + 1);
        int cntAB = 0, cntBA = 0;
        for(int i = 1; i <= n - 1; i++) {
            if(s[i] == 'a' && s[i + 1] == 'b') {
                cntAB++;
            }
            if(s[i] == 'b' && s[i + 1] == 'a') {
                cntBA++;
            }
        }
        if(cntAB != cntBA) {
            if(s[1] == 'a') {
                s[1] = 'b';
            } else {
                s[1] = 'a';
            }
        }
        printf("%s\n", s + 1);
    }

    return 0;
}

```

# B - Update Files

## 题意

有 $n$ 台计算机，$k$ 条数据线，每条数据线同一时刻只能连接两台计算机，问最少要几轮才能将第 $1$ 台计算机上的文件传输到全部 $n$ 台计算机上。

## 思路

当目前有文件的计算机总数少于 $k$ 时，传输文件的计算机总数显然会每次翻倍，即 $1$, $2$, $4$, $8$, $16$…… 但如果当前有文件的计算机总数超过 $k$，就只能每次增加 $k$ 个，最后结果为 $\lceil \frac{n - 2^m}{k} \rceil + m$ ($2^m \leq n$ 且 $2^m > k$，$2^{m - 1} \leq k$)。

## 代码

```cpp
#include<cstdio>
using namespace std;

long long n, k;

int main() {
    int T;

    scanf("%d", &T);
    while(T--) {
        scanf("%lld%lld", &n, &k);
        if(n == 1) {
            printf("0\n");
            continue;
        }
        if(k == 1) {
            printf("%lld\n", n - 1);
            continue;
        }
        long long b = 1, cnt = 0;
        while(b <= k && b * 2 <= n) {
            b *= 2;
            cnt++;
        }
        cnt += (n - b) / k;
        if((n - b) % k != 0) {
            cnt++;
        }
        printf("%lld\n", cnt);
    }

    return 0;
}
```

# C - Banknotes

## 题意

给出 $n$ 种面值为 $10^{a_i}$ 的纸币，求最少的金额使其无法用 $k$ 张纸币表示出来。

## 思路

可以转换为找到最小的必须用 $k + 1$ 种纸币表示出的金额，定义 $s = k + 1$。对于第 $i$ 种纸币，令 $s$ 减去 $10^{a_{i + 1} - a_i} - 1$，如果不能减去或者已经到了面值最大的纸币，就直接输出当前的数，并在后面用 $a_i$ 个 $9$ 填充。

## 代码

```cpp
#include<cstdio>
#include<cstring>
using namespace std;

const int MAXN = 10;
const int base[10] = {1, 10, 100, 1000, 10000, 100000, 1000000, 10000000, 100000000, 1000000000};

bool vis[MAXN];
int a[MAXN], ans[MAXN];

int main() {
    int T;

    scanf("%d", &T);
    while(T--) {
        int n, k;

        scanf("%d%d", &n, &k);
        memset(vis, 0, sizeof(vis));
        for(int i = 1; i <= n; i++) {
            scanf("%d", &a[i]);
            vis[a[i]] = 1;
        }
        int s = k + 1, p = n;
        for(int i = 1; i <= n; i++) {
            if(i == n) {
                ans[i] = s;
                break;
            }
            if(s > base[a[i + 1] - a[i]] - 1) {
                ans[i] = base[a[i + 1] - a[i]] - 1;
                s -= base[a[i + 1] - a[i]] - 1;
            } else {
                ans[i] = s;
                p = i;
                break;
            }
        }
        printf("%d", ans[p]);
        for(int i = 1; i <= a[p]; i++) {
            printf("9");
        }
        printf("\n");
    }

    return 0;
}
```

# E - Arena

## 题意

竞技场里有 $n$ 个英雄，每个英雄的生命值为 $a_i$（$1 \leq a_i \leq x$）。每一次战斗中，每个英雄都会对其他所有英雄造成 $1$ 点伤害，生命值降到 $0$ 后英雄就会退场，如果若干次战斗后只剩下一个英雄，这个英雄就是赢家。

求有多少种生命值方案使得最后没有赢家，答案模 $998244353$。两种方案不同当且仅当至少有一个英雄的生命值不同。

## 思路

直接思考没有赢家的方案数似乎没什么思路，我们不妨考虑计算有赢家的方案数。

定义 $f_{i, j}$ 为有 $i$ 个英雄，生命值最大为 $j$ 的方案数，就可以得到状态转移方程：

$$f_{i, j} = \sum_{k = 0}^{i - 1}f_{i - k, j - i + 1}(i - 1)^k C_i^k$$

其中 $k$ 为这一次战斗退场的英雄数。

要注意的一点是，这道题的时限会导致一般的快速幂不能过。我采取的方式是对幂进行预处理，具体内容参见代码。

## 代码

```cpp
#include<cstdio>
#include<algorithm>
using namespace std;

const int MAXN = 500 + 5;
const int MOD = 998244353;

long long C[MAXN][MAXN], f[MAXN][MAXN], qpow[MAXN][MAXN];
int n, m;

void init() {
	C[0][0] = 1;
	for(int i = 1; i <= n; i++) {
		C[i][0] = 1;
		for(int j = 1; j <= i; j++) {
			C[i][j] = (C[i - 1][j - 1] + C[i - 1][j]) % MOD;
		}
	}
	for(int i = 1; i < MAXN; i++) {
		qpow[i][0] = 1;
		for(int j = 1; j < MAXN; j++) {
			qpow[i][j] = qpow[i][j - 1] * i % MOD;
		}
	}
}

int main() {
	scanf("%d%d", &n, &m);
	init();
	for(int j = 1; j <= m; j++) {
		f[1][j] = j;
		f[2][j] = j * (j - 1);
	}
	for(int i = 3; i <= n; i++) {
		for(int j = i; j <= m; j++) {
			for(int k = 0; k < i; k++) {
				f[i][j] = (f[i][j] + (f[i - k][j - i + 1] * qpow[i - 1][k]) % MOD * C[i][k] % MOD) % MOD;
			}
		}
	}
	printf("%lld\n", ((qpow[m][n] - f[n][m]) % MOD + MOD) % MOD);
	
	return 0;
}

```

（剩下的题等会就补）

（咕咕咕）