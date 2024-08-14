---
date:
  created: 2024-06-17
  updated: 2024-06-17
draft: false
pin: false
comments: true

categories:
  - 做题记录
tags:
  - 区间DP基础
---

## BNDSOJ DP-3-2 能量项链

[BNDSOJ DP-3-2 能量项链](https://onlinejudge.bnds.cn/contest/problem?id=714&pid=1)

发现给定珠子序列为长度为 $n$ 的一个环，那么显然需要破环成链；具体的，将整个序列复制一份接到原序列末尾，区间 DP 已经保证了统计的区间长度不会超过 $n$，统计答案时枚举起点取最大值即可

不妨依据题意设 $dp[i][j]$ 表示将区间 $[i, j]$ 间的珠子合并所能得到的最大能量；设第 $i$ 颗珠子上的值为 $v[i]$，考虑转移，则枚举 $k(i<k<j)$ 作为 "分割点"，$dp[i][j] = max(dp[i][k]+dp[k][j]+v[i] \times v[k] \times v[j])$，即先分别统计将 $[i, k]$ 与 $[k, j]$ 合并能获得的价值的最大值，最终加上本次合并获得的价值即 $v[i] \times v[k] \times v[j]$

时间复杂度 $O(n^3)$

```cpp
#include<bits/stdc++.h>
using namespace std;
const int MAXN = 205;

int n, ans=0;
int num[MAXN];
int dp[MAXN][MAXN];

int main(){
	scanf("%d", &n);
	
	for (int i=1; i<=n; i++){
		scanf("%d", &num[i]);
		num[n+i] = num[i];
	}
	
	for (int l=1; l<=n; l++){
		for (int i=1; i<=2*n-l; i++){
			int lhs=i, rhs=i+l;
			for (int k=lhs+1; k<rhs; k++){
				dp[lhs][rhs] = max(dp[lhs][rhs], dp[lhs][k]+dp[k][rhs]+num[lhs]*num[k]*num[rhs]);
			}
		}
	}
	
	for (int i=1; i<=n; i++){
		ans = max(ans, dp[i][i+n]);
	}
	printf("%d", ans);
	return 0;
}
```

## BNDSOJ DP-3-3 乘法游戏

[BNDSOJ DP-3-3 乘法游戏](https://onlinejudge.bnds.cn/contest/problem?id=714&pid=2)

发现题目中要求的并不是合并牌而是拿走牌，因此满足区间 DP 的性质，考虑设 $dp[i][j]$ 表示将区间 $[i, j]$ 内的数合并能得到的最大价值，设第 $i$ 个数为 $v[i]$；考虑转移，枚举转移点 $k(i<k<j)$，则 $dp[i][j] = max(dp[i][k]+dp[k][j]+v[i] \times v[k] \times v[j])$；

初始状态为 $dp[i][i]=0(1 \le i \le n)$ 与 $dp[i][i+1]=0(1 \le i < n)$，因为无法合并长度小于 $3$ 的区间；题目中要求不能选取第 $1$ 个与第 $n$ 个数，但区间 DP 的转移点一定满足在 $(1, n)$ 范围内，不会越出，因此不需要特殊考虑

答案即为 $dp[1][n]$

时间复杂度 $O(n^3)$

```cpp
#include<bits/stdc++.h>
using namespace std;
const int MAXN = 205;

int n;
int num[MAXN];
int dp[MAXN][MAXN];

int main(){
	scanf("%d", &n);
	memset(dp, 0x3f, sizeof(dp));
	
	for (int i=1; i<=n; i++){
		scanf("%d", &num[i]);
		dp[i][i] = 0;
	}
	for (int i=1; i<n; i++){
		dp[i][i+1] = 0;
	}
	
	for (int l=1; l<=n; l++){
		for (int i=1; i<=n-l+1; i++){
			int lhs=i, rhs=i+l-1;
			for (int k=lhs+1; k<rhs; k++){
				dp[lhs][rhs] = min(dp[lhs][rhs], dp[lhs][k]+dp[k][rhs]+num[lhs]*num[k]*num[rhs]);
			}
		}
	}
	
	printf("%d", dp[1][n]);
	return 0;
}
```

## BNDSOJ DP-3-4 石子合并

[BNDSOJ DP-3-4 石子合并](https://onlinejudge.bnds.cn/contest/problem?id=714&pid=3)

由于最大值与最小值本质相同，以下只考虑最大值

首先注意到石子排成了一个环，因此还是考虑破环成链，将原序列复制一份接在原序列的后面；设第 $i$ 堆石子价值为 $v[i]$，合并区间 $[i, j]$ 的石子的价值为 $val(i, j)$，考虑设 $dp[i][j]$ 表示将区间 $[i, j]$ 中的石子合并能获取的最大价值，仍然枚举 $k(i \le k<j)$ 作为转移点，$dp[i][j] = max(dp[i][k]+dp[k+1][j]+val(i, j))$；由于一次合并获得的是两堆石子的价值之和，合并一次区间就获得一次区间内所有石子的价值之和，因此 $val(i, j) = \sum_{k=i}^{j} v[k]$，可以使用前缀和维护

对于初始状态，由于一次合并最少合并相邻的两堆石子，因此 $dp[i][i]=0 (1 \le i \le n)$

时间复杂度 $O(n^3)$

```cpp
#include<bits/stdc++.h>
using namespace std;
const int MAXN = 405;

int n, ans=0, ans2=2147483647;
int num[MAXN], sum[MAXN];
int dp[MAXN][MAXN], dp2[MAXN][MAXN];

int main(){
	scanf("%d", &n);
	memset(dp2, 0x3f, sizeof(dp2));
	
	for (int i=1; i<=n; i++){
		scanf("%d", &num[i]);
		num[n+i] = num[i];
	}
	for (int i=1; i<=2*n; i++){
		sum[i] = sum[i-1]+num[i];
		dp2[i][i] = 0;
	}
	
	for (int l=2; l<=n; l++){
		for (int i=1; i<=2*n-l-1; i++){
			int lhs=i, rhs=i+l-1;
			for (int k=lhs; k<rhs; k++){
				dp[lhs][rhs] = max(dp[lhs][rhs], dp[lhs][k]+dp[k+1][rhs]+sum[rhs]-sum[lhs-1]);
				dp2[lhs][rhs] = min(dp2[lhs][rhs], dp2[lhs][k]+dp2[k+1][rhs]+sum[rhs]-sum[lhs-1]);	
			}
		}	
	}
	
	for (int i=1; i<=n-1; i++){
		ans = max(ans, dp[i][i+n-1]);
		ans2 = min(ans2, dp2[i][i+n-1]);
	}
	printf("%d\n%d", ans2, ans);
	return 0;
}
```

## BNDSOJ DP-3-5 括号配对

[BNDSOJ DP-3-5 括号配对](https://onlinejudge.bnds.cn/contest/problem?id=714&pid=4)

令第 $i$ 个字符为 $s[i]$，设 $dp[i][j]$ 表示将区间 $[i, j]$ 变为 $GBE$ 的最少操作数，仍然考虑枚举 $k(i \le k<j)$，由于区间中的同一位置只需要修改一次，因此 $dp[i][j]=min(dp[i][k]+dp[k+1][j])$；注意第二个区间的左端点为 $k+1$，枚举 $k$ 的范围包含 $i$；

但如果这么转移，实际上忽略了 "如果 $A$ 是 $GBE$，那么 $[A]$ 与 $(A)$ 也是 $GBE$" 的条件，因此可以很容易举出反例，比如输入 `[(]`，会输出 `3`，实际上应该输出 `1`；那么如何加上这一条件？当我们合并区间 $[i, j]$ 时，如果 $s[i]$ 与 $s[j]$ 构成了 `[]` 或者 `()`，就只需要考虑合并 $[i+1, j-1]$，因此在转移后判断 $s[i]$ 与 $s[j]$ 是否构成 `[]` 与 `()`，如果构成就加上 $dp[i][j] = min(dp[i][j], dp[i+1][j-1]) $ 即可

初始条件为 $dp[i][i] = 1 (1 \le i \le n)$，答案即为 $dp[i][n]$

时间复杂度 $O(n^3)$

```cpp
#include<bits/stdc++.h>
using namespace std;
const int MAXN = 505, MAXINT = 2e9;

string s;
int dp[MAXN][MAXN];

int main(){
	cin >> s;
	
	for (int i=0; i<s.size(); i++){
		dp[i+1][i+1] = 1;
	}
	
	for (int l=2; l<=s.size(); l++){
		for (int i=1; i<=s.size()-l+1; i++){
			int lhs=i, rhs=i+l-1;
			dp[lhs][rhs] = MAXINT;
			for (int k=lhs; k<rhs; k++){
				dp[lhs][rhs] = min(dp[lhs][rhs], dp[lhs][k]+dp[k+1][rhs]);
			}
			
			if ((s[lhs-1] == '(' && s[rhs-1] == ')') || (s[lhs-1] == '[' && s[rhs-1] == ']')){
				dp[lhs][rhs] = min(dp[lhs][rhs], dp[lhs+1][rhs-1]);
			}
		}	
	}
	
	printf("%d", dp[1][s.size()]);
	return 0;
}
```

## BNDSOJ DP-4-2 凸多边形划分

[BNDSOJ DP-4-2 凸多边形划分](https://onlinejudge.bnds.cn/contest/problem?id=760&pid=1#problem-anchor)

我们考虑设 $dp[i][j]$ 表示将以 $i$ 为开头，$j$ 为结尾的多边形分割后可以获得的最大价值；注意到每个边都肯定会用且仅用一次，因此我们枚举分割点 $k (i<k<j)$，使得 $i, j, k$ 构成一个三角形，再套路地统计 $dp[i][k]$ 与 $dp[k][j]$ 即可；转移方程即为 $dp[i][j] = min(dp[i][j], dp[i][k]+dp[k][j]+num[i] \times num[k] \times num[j])$

值域较大，$long long$ 存不下，记得开 $int128$

时间复杂度 $O(n^3)$

```cpp
#include<bits/stdc++.h>
using namespace std;
const int MAXN = 55;
const __int128 INF = 1e35;

__int128 read128(){
	__int128 x=0;bool f=1;char ch=getchar();
	while (ch<'0'||ch>'9'){if(ch=='-') f=!f;ch=getchar();}
	while (ch>='0'&&ch<='9'){x=(x<<1)+(x<<3)+(ch^48);ch=getchar();}
	x=(f?x:-x);return x;
}
inline void put128(__int128 x){
	if (x<0) putchar('-'), x=-x;
	if (x>9) put128(x/10);
	putchar(x%10+'0'); return;
}

int N;
__int128 num[MAXN];
__int128 dp[MAXN][MAXN];

int main(){
	scanf("%d", &N);
	
	for (int i=1; i<=N; i++){
		num[i]=read128();
	}
	
	for (int len=3; len<=N; len++){
		for (int i=1; i<=N-len+1; i++){
			int lhs=i, rhs=i+len-1;
			dp[lhs][rhs] = INF;
			for (int k=lhs+1; k<rhs; k++){
				dp[lhs][rhs] = min(dp[lhs][rhs], dp[lhs][k]+dp[k][rhs]+num[lhs]*num[k]*num[rhs]);
			}
		}
	}
	
	put128(dp[1][N]);
	return 0;
}
```