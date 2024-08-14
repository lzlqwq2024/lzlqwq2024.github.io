---
date:
  created: 2024-05-06
  updated: 2024-05-06
draft: false
pin: false
comments: true

categories:
  - 做题记录
tags:
  - DP基础
---

## BNDSOJ DP-1-1 最长不下降子序列

[BNDSOJ DP-1-1 最长不下降子序列](https://onlinejudge.bnds.cn/contest/problem?id=712&pid=0)

对于序列 $a_i$，设 $f[i]$ 表示以 $i$ 结尾的最长不下降子序列长度，循环枚举 $j$ $(1 \leq j \leq i-1)$，若 $a_j \leq a_i$，则可以从 $f[j]$ 转移至 $f[i]$，具体来说 $f[i] = f[j]+1$

时间复杂度 $O(n^2)$

```cpp
#include<bits/stdc++.h>
using namespace std;
const int MAXN = 10005;

int n, ans=0;
int num[MAXN], f[MAXN];

int main(){
	scanf("%d", &n);
	
	for (int i=1; i<=n; i++){
		scanf("%d", &num[i]);
	}
	for (int i=1; i<=n; i++){
		f[i] = 1;
	}
	
	for (int i=1; i<=n; i++){
		for (int j=1; j<i; j++){
			if (num[j] <= num[i]){
				f[i] = max(f[i], f[j]+1);
			}
		}
	}
	
	for (int i=1; i<=n; i++){
		ans = max(ans, f[i]);
	}
	
	printf("%d", ans);
	return 0;
}
```


## BNDSOJ DP-1-2 友好城市

[BNDSOJ DP-1-2 友好城市](https://onlinejudge.bnds.cn/contest/problem?id=712&pid=1#problem-anchor)

使用结构体记录对应的友好城市，首先按照河岸一边城市的横坐标 $a_i$ 进行排序

设两对友好城市横坐标为 $(a_1, b_1)$ 与 $(a_2, b_2)$，若希望使得两对友好城市不 "相交"，当且仅当 ($a_1 \leq a_2$ 且 $ b_1 \leq b_2$) 或 ($a_1 \geq a_2$ 且 $b_1 \geq b_2$)

因此，希望排序后河岸另一边城市的横坐标 $b_i$ 单调不降，问题即转化为对序列 $b_i$ 求最长不下降子序列

时间复杂度 $O(n^2)$

```cpp
#include<bits/stdc++.h>
using namespace std;
const int MAXN = 5005;

int N, ans=0;
int f[MAXN];

struct City{
	int x;
	int y;
}pos[MAXN];

bool cmp(City a, City b){
	return a.x < b.x;
}

int main(){
	scanf("%d", &N);
	
	for (int i=1; i<=N; i++){
		scanf("%d%d", &pos[i].x, &pos[i].y);
	}
	
	sort(pos+1, pos+N+1, cmp);
	
	for (int i=1; i<=N; i++){
		f[i] = 1;
	}
	
	for (int i=1; i<=N; i++){
		for (int j=1; j<i; j++){
			if (pos[j].y <= pos[i].y){
				f[i] = max(f[i], f[j]+1);
			}
		}
	}
	
	for (int i=1; i<=N; i++){
		ans = max(ans, f[i]);
	}
	
	printf("%d", ans);
	return 0;
}
```

## BNDSOJ DP-1-4 贝茜的晨练计划

[BNDSOJ DP-1-4 贝茜的晨练计划](https://onlinejudge.bnds.cn/contest/problem?id=712&pid=3#problem-anchor)

首先考虑使用 $dp[i][j]$ 表示第 $i$ 分钟疲劳度为 $j$ 能跑的最远距离，但是注意到贝茜一旦开始休息就必须休息到疲劳度为 $0$，状态不好转移

由于休息到疲劳度为 $0$ 的条件很强，考虑特化状态，令 $dp[i]$ 表示第 $i$ 分钟且疲劳度为 $0$ 时能跑的最远距离，显然答案为 $dp[N]$

考虑转移，首先可以由上一分钟疲劳度就为 $0$ 转移得来，相当于上一分钟选择休息，没有多跑任何距离，此时转移方程为 $dp[i] = dp[i-1]$

其次，可以在该分钟前时间差大于 $1$ 的位置转移过来；具体的，由于复杂度从 $0$ 变成 $0$，一定是跑了一段时间又持续休息了一段时间，而这两段时间显然相等；为方便枚举，不妨设当前时间为 $i$，这段时间为 $j\  (j<=min(i/2, M))$，由于贝茜在第 $i-2 \times j+1$ 到第 $i-j$ 分钟选择了跑步，因此转移方程为 $dp[i] = dp[i-2 \times j]+ \sum_{k=i-2 \times j+1}^{i-j} D[k]$，显然可以用前缀和维护

时间复杂度 $O(NM)$

```cpp
#include<bits/stdc++.h>
using namespace std;
const int MAXN = 1e4+5;

int N, M;
int f[MAXN], D[MAXN], sum[MAXN];

int main(){
	scanf("%d%d", &N, &M);
	
	for (int i=1; i<=N; i++){
		scanf("%d", &D[i]);
		sum[i] = sum[i-1]+D[i];
	}
	
	for (int i=1; i<=N; i++){
		f[i] = max(f[i], f[i-1]);
		
		for (int j=1; j<=min(i/2, M); j++){
			f[i] = max(f[i], f[i-2*j]+sum[i-j]-sum[i-2*j]);
		}
	}
	
	printf("%d", f[N]);
	return 0;
}
```

## BNDSOJ DP-1-5 ZBRKA

[BNDSOJ DP-1-5 ZBRKA](https://onlinejudge.bnds.cn/contest/problem?id=712&pid=4)

对于一个长度为 $n-1$ 的 $1$ 到 $n-1$ 的排列，设其逆序对数为 $C$，注意到我们想要往排列中插入数 $n$ 形成新的排列，又因为 $n$ 比原排列中的所有数都大，因此可以新形成 $0$ 到 $n-1$ 个逆序对，即对长度为 $n$ 的逆序对数为 $C$ 到 $C+n-1$ 的排列的数量贡献都为 $1$

具体的，设 $dp[i][j]$ 表示长度为 $i$ 的且逆序对数量为 $j$ 个的排列个数，转移方程为 $dp[i][j] = \sum_{k=j-(i-1)}^{j} dp[i-1][k]$，显然可以使用前缀和优化，可以压掉一维，但注意每次在转移之前需要先实时更新前缀和数组，同时赋值 $dp[0] = sum[0] = 1$，因为逆序对数为 $0$ 时也有一种方案满足 (即 $1\ 2\ 3\ ...\ n$)

时间复杂度 $O(NC)$

```cpp
#include<bits/stdc++.h>
using namespace std;
#define int long long
const int MAXC = 1e5+5, MOD = 1e9+7;

int N, C;
int f[MAXC], sum[MAXC];

signed main(){
	scanf("%lld%lld", &N, &C);
	
	for (int i=2; i<=N; i++){
		f[0] = sum[0] = 1;
		for (int j=1; j<=C; j++){
			sum[j] = (sum[j-1]+f[j])%MOD;
		}
		for (int j=0; j<=C; j++){
			if (j >= i){
				f[j] = (sum[j] - sum[j-i])%MOD;
			}
			else{
				f[j] = sum[j]%MOD;
			}
		}
	}
	
	if (f[C] < 0){
		printf("%lld", MOD+f[C]);
		return 0;	
	}
	printf("%lld", f[C]);
	return 0;
}
```

## BNDSOJ DP-1-8 潜水员

[BNDSOJ DP-1-8 潜水员](https://onlinejudge.bnds.cn/contest/problem?id=712&pid=7)

注意到选择一个气缸会增加两个变量，及氧和氮的数量，因此这是一个二维费用背包，令 $dp[k][i][j]$ 表示前 $k$ 个气缸中氧气为数量 $i$ 且氮气数量为 $j$ 时总共的最小重量

由于希望气缸总重最小，给 $dp$ 数组初始赋极大值，同时注意初始化 $dp[0][0] = 0$，令氧气数量为 $a[i]$，氮气数量为 $b[i]$，气缸重量为 $v[i]$，则转移方程为 $dp[k][i][j] = min(dp[k-1][i][j], dp[k-1][i-a[k]][j-b[k]]+v[k])$，第一维可以压掉

注意转移时，不一定氧的数量与氮的数量的上界一定是 $m$ 和 $n$，因此设定一个偏移量 $DIF = 80$，氧和氮的上界分别为 $m+DIF$ 与 $n+DIF$

注意枚举答案时要枚举到所装多于 $m$ 与 $n$ 时的状态，取最大值

时间复杂度 $O(kmn)$

```cpp
#include<bits/stdc++.h>
using namespace std;
const int MAXK = 1e4+5, MAXQ = 1005, DIF = 80;

int m, n, ans=2147483647;
int k;
int a[MAXK], b[MAXK], c[MAXK];
int dp[MAXQ][MAXQ];

int main(){
	memset(dp, 0x7f, sizeof(dp));
	dp[0][0] = 0;
	scanf("%d%d", &m, &n);
	scanf("%d", &k);
	
	for (int i=1; i<=k; i++){
		scanf("%d%d%d", &a[i], &b[i], &c[i]);
	}
	
	for (int l=1; l<=k; l++){
		for (int i=m+DIF; i>=a[l]; i--){
			for (int j=n+DIF; j>=b[l]; j--){
				dp[i][j] = min(dp[i][j], dp[i-a[l]][j-b[l]]+c[l]);
			}
		}
	}
	
	for (int i=m; i<=m+DIF; i++){
		for (int j=n; j<=n+DIF; j++){
			ans = min(ans, dp[i][j]);
		}
	}
	
	printf("%d", ans);
	return 0;
}
```

## BNDSOJ DP-1-10 金明的预算方案

[BNDSOJ DP-1-10 金明的预算方案](https://onlinejudge.bnds.cn/contest/problem?id=712&pid=9)

对于每个主件及其的配件，只有 买主件、买主件和 $1$ 个附件、买主件和 $2$ 个附件三种购买方案，且只能选择一种，因此可以视为分组背包

注意价值为 $v[i] \times w[i]$ 而非 $v[i]$

时间复杂度 $O(NM)$

```cpp
#include<bits/stdc++.h>
using namespace std;
const int MAXT = 505, MAXN = 1e5+5;

int N, M, T=0;
int cnt[MAXT], belong[MAXT];
int w[MAXT][MAXT], v[MAXT][MAXT];
int f[MAXN];

int main(){
	scanf("%d%d", &N, &M);
	
	for (int i=1; i<=M; i++){
		int price, value, type;
		scanf("%d%d%d", &price, &value, &type);
		if (type == 0){
			T++;
			cnt[T]++;
			belong[i] = T;
			w[T][cnt[T]] = price;
			v[T][cnt[T]] = price*value;
		}
		else{
			cnt[belong[type]]++;
			w[belong[type]][cnt[belong[type]]] = w[belong[type]][cnt[belong[type]]-1]+price;
			v[belong[type]][cnt[belong[type]]] = v[belong[type]][cnt[belong[type]]-1]+price*value;
		}	
	}
	
	/*
	for (int i=1; i<=T; i++){
		cout << "object " << i << endl;
		cout << "main weight: " << w[i][1] << " value: " << v[i][1] << endl;
		cout << "side1 weight: " << w[i][2] << " value: " << v[i][2] << endl;
		cout << "side2 weight: " << w[i][3] << " value: " << v[i][3] << endl;
	}
	*/
	
	for (int k=1; k<=T; k++){
		for (int i=N; i>=0; i--){
			for (int j=1; j<=cnt[k]; j++){
				if (i >= w[k][j]){
					f[i] = max(f[i], f[i-w[k][j]]+v[k][j]);
				}
			}
		}
	}
	
	printf("%d", f[N]);
	return 0;
}
```

## BNDSOJ DP-2-2 橱窗布置

[BNDSOJ DP-2-2 橱窗布置](https://onlinejudge.bnds.cn/contest/problem?id=713&pid=1&_pjax=%23p0)

定义状态 $dp[i][j]$ 表示前 $i$ 朵花中，第 $i$ 朵花放在第 $j$ 个花瓶时美学值的最大值

考虑转移，注意到我们已经固定了 $i$ 的位置，因此不难想到进一步固定 $i-1$ 的位置，进行转移；由于第 $i-1$ 朵花至少放在第 $i-1$ 个花瓶中，至多放在第 $j-1$ 个花瓶中，因此转移方程为 $dp[i][j] = \sum_{k=i-1}^{j-1} max(dp[i-1][k]+beauty[i][j])$ ($beauty[i][j]$ 表示第 $i$ 朵花在第 $j$ 个花瓶中的美学值)

由于还需要记录方案，考虑用 $from[i][j]$ 保存一个 $pair<int, int>$，表示其由哪个状态转移过来，递归输出即可

时间复杂度 $O(FV^2)$

```cpp
#include<bits/stdc++.h>
using namespace std;
const int MAXF = 505, MAXV = 505;

int F, V, ans=-2147483647, final=-1;
int beauty[MAXF][MAXV];
int dp[MAXF][MAXV];
pair <int, int> from[MAXF][MAXV];

void getans(int step, int pos){
	if (step == 1){
		printf("%d ", pos);
		return;
	}
	// cout << "debug: " << pos << " " << from[pos].second << endl;
	getans(from[step][pos].first, from[step][pos].second); 
	printf("%d ", pos);
}

int main(){
	scanf("%d%d", &F, &V);
	memset(dp, -0x3f, sizeof(dp));
	dp[0][0] = 0;
	
	for (int i=1; i<=F; i++){
		for (int j=1; j<=V; j++){
			scanf("%d", &beauty[i][j]);
		}
	}
	
	for (int i=1; i<=F; i++){
		for (int j=i; j<=V-(F-i); j++){
			for (int k=i-1; k<=j-1; k++){
				if (dp[i-1][k]+beauty[i][j] > dp[i][j]){
					dp[i][j] = dp[i-1][k]+beauty[i][j];	
					from[i][j] = {i-1, k};
				}
			}
		}
	}
	
	for (int i=F; i<=V; i++){
		if (dp[F][i] > ans){
			ans = max(ans, dp[F][i]);
			final = i;
		}
	}
	printf("%d\n", ans);
	getans(F, final);
}
```

## BNDSOJ DP-2-3 花匠

[BNDSOJ DP-2-3 花匠](https://onlinejudge.bnds.cn/contest/problem?id=713&pid=2)

题目要求的实际上是一个交替增减的子序列，由于 $O(n^2)$ 的时间复杂度会超时，因此考虑只从序列中的上一位转移状态，因此发现实际上只需要考虑序列变化的趋势，即如果趋势变化则贡献 $+1$，若不变化则贡献也不变

因此设 $dp[i][0/1]$ 表示在原序列第 $i$ 位变化趋势为 $0/1$ ($0$ 表示当前变化趋势为上升，$1$ 表示当前变化趋势为下降) 的留下的花的最大值；设 $a_i$ 为原序列的第 $i$ 项，分情况讨论，当 $a_i > a_{i-1}$ 时，$dp[i][0] = max(dp[i-1][0], dp[i-1][1]+1)$ 且 $dp[i][1] = dp[i-1][1]$，当 $a_i < a_{i-1}$ 时，$dp[i][1] = max(dp[i-1][1], dp[i-1][0]+1)$ 且 $dp[i][0] = dp[i-1][0]$，当 $a_i = a_{i-1}$ 时，$dp[i][0] = dp[i-1][0]$ 且 $dp[i][1] = dp[i-1][1]$

边界条件显然为 $dp[1][0] = dp[1][1] = 1$，输出时直接输出 $max(dp[n][0], dp[n][1])$ 即可，$n$ 为原序列长度，本质上实际是递推

时间复杂度 $O(n)$

```cpp
#include<bits/stdc++.h>
using namespace std;
const int MAXN = 1e5+5;

int n, ans=0;
int num[MAXN];
int dp[MAXN][2];

int main(){
	scanf("%d", &n);
	
	for(int i=1; i<=n; i++){
		scanf("%d", &num[i]);
	}
	
	dp[1][0] = dp[1][1] = 1;
	 
	for (int i=2; i<=n; i++){
		if (num[i] > num[i-1]){
			dp[i][0] = max(dp[i-1][0], dp[i-1][1]+1);
			dp[i][1] = dp[i-1][1];
		}
		else if (num[i] < num[i-1]){
			dp[i][1] = max(dp[i-1][1], dp[i-1][0]+1);
			dp[i][0] = dp[i-1][0];
		}
		else{
			dp[i][0] = dp[i-1][0];
			dp[i][1] = dp[i-1][1];
		}
	}
	
	printf("%d", max(dp[n][0], dp[n][1]));
	return 0;
}
```

## BNDSOJ DP-2-4 编辑距离

设 $dp[i][j]$ 为将 $A$ 串前 $i$ 位变为 $B$ 串前 $j$ 位所需的最小代价，考虑由 $i-1$ 与 $j-1$ 转移而来，由于支持修改、添加和删除，实际上有如下几种方式

1. 将 $A$ 串前 $i-1$ 个变为 $B$ 串前 $j-1$ 个，再将 $A_i$ 修改为 $B_j$ 即可，分两种情况，如果 $A_i = B_j$ 则 $dp[i][j] = dp[i-1][j-1]+1$，否则 $dp[i][j] = dp[i-1][j-1]+1$
2. 将 $A$ 串前 $i-1$ 个变为 $B$ 串前 $j$ 个，再将 $A_i$ 删除即可，$dp[i][j] = dp[i-1][j]+1$
3. 将 $A$ 串前 $i$ 个变为 $B$ 串前 $j-1$ 个，再在 $A$ 中插入 $B_j$ 即可，$dp[i][j] = dp[i][j-1]+1$

设 $A$ 的长度为 $n$，$B$ 的长度为 $m$，则初始情况为 $dp[0][i] = i(1<=i<=m)$，即将空串变为 $B$ 的前 $i$ 个字符，需要插入 $i$ 次；$dp[i][0] = i(1<=i<=n)$，即将 $A$ 的前 $i$ 个字符变为空串，需要删除 $i$ 次；最终答案即为 $dp[n][m]$

时间复杂度 $O(nm)$，$n$ 与 $m$ 为串长

```cpp
#include<bits/stdc++.h>
using namespace std;
const int MAXN = 5005;
string a, b;

int dp[MAXN][MAXN];

int main(){
	cin >> a >> b;
	
	for (int i=0; i<a.size(); i++){
		dp[i+1][0] = i+1;
	}
	for (int i=0; i<b.size(); i++){
		dp[0][i+1] = i+1;
	}
	
	for (int i=0; i<a.size(); i++){
		for (int j=0; j<b.size(); j++){
			if (a[i] == b[j]){
				dp[i+1][j+1] = min(dp[i][j], min(dp[i][j+1]+1, dp[i+1][j]+1));
			}
			else{
				dp[i+1][j+1] = min(dp[i][j]+1, min(dp[i][j+1]+1, dp[i+1][j]+1));	
			}
		}
	}
	
	cout << dp[a.size()][b.size()];
	return 0;
}
```

## BNDSOJ DP-2-5 乌龟棋

[BNDSOJ DP-2-5 乌龟棋](https://onlinejudge.bnds.cn/contest/problem?id=713&pid=4)

首先考虑设 $dp[i][a][b][c][d]$ 表示走到第 $i$ 个格子用了 $a$ 张 $1$ 卡片，$b$ 张 $2$ 卡片，$c$ 张 $3$ 卡片，$d$ 张 $4$ 卡片，但是这样 $dp$ 会超时；我们注意到一个很关键的点，只需要各用了几张不同类型的卡片，实际上就已经可以算出当前到的格子位置，因此第一维实际上是没有用的

设 $dp[a][b][c][d]$ 为 (走到当前格子) 四种卡分别用了多少张时能获得价值的最大值，显然当前格子位置为 $chart = a \times 1 + b \times 2 + c \times 3 + d \times 4$，因为可以通过在距离当前格子前 $1/2/3/4$ 个位置的格子使用 一个 $a/b/c/d$ 类型的卡片从而前进到当前格子，因此转移方程即为 $dp[a][b][c][d] = max(dp[a-1][b][c][d], dp[a][b-1][c][d], dp[a][b][c-1][d], dp[a][b][c][d-1])+val[chart]$，其中 $val[i]$ 为在第 $i$ 个格子可以获得的价值；注意 $a/b/c/d = 0$ 时不能再由 $a-1/b-1/c-1/d-1$ 转移，需要特殊考虑

初始情况为 $dp[0][0][0][0] = val[1]$，因为题目中说了自动获得起点位置的价值

时间复杂度 $O(A \times B \times C \times D)$，$A/B/C/D$ 为四种卡片的个数

```cpp
#include<bits/stdc++.h>
using namespace std;
const int MAXN = 405;

int N, M;
int num[MAXN], cnt[5];
int dp[45][45][45][45];

int main(){	
	scanf("%d%d", &N, &M);
	
	for (int i=1; i<=N; i++){
		scanf("%d", &num[i]);	
	}
	for (int i=1; i<=M; i++){
		int val;
		scanf("%d", &val);
		cnt[val]++;
	}
	
	dp[0][0][0][0] = num[1];
	
	for (int i=0; i<=cnt[1]; i++){
		for (int j=0; j<=cnt[2]; j++){
			for (int k=0; k<=cnt[3]; k++){
				for (int l=0; l<=cnt[4]; l++){
					int chart = i*1+j*2+k*3+l*4+1;
					if (i!=0){
						dp[i][j][k][l] = max(dp[i][j][k][l], dp[i-1][j][k][l]+num[chart]);
					}
					if (j!=0){
						dp[i][j][k][l] = max(dp[i][j][k][l], dp[i][j-1][k][l]+num[chart]);
					}
					if (k!=0){
						dp[i][j][k][l] = max(dp[i][j][k][l], dp[i][j][k-1][l]+num[chart]);
					}
					if (l!=0){
						dp[i][j][k][l] = max(dp[i][j][k][l], dp[i][j][k][l-1]+num[chart]);
					}
				}
			}
		}
	}
	
	printf("%d", dp[cnt[1]][cnt[2]][cnt[3]][cnt[4]]);
	return 0;
}
```

## BNDSOJ DP-2-6 传纸条

[BNDSOJ DP-2-6 传纸条](https://onlinejudge.bnds.cn/contest/problem?id=713&pid=5)

由于有两路传纸条路径不能相交的限制，直接跑两次最大值 $dp$ 不能保证正确性；

考虑设 $dp[i_1][j_1][i_2][j_2]$ 为第一路纸条传到 $(i_1, j_1)$ 且第二路纸条传到 $(i_2, j_2)$ 时的最大值；关键在于使得两条路线不会相交，观察到两条路线必然一条在 "右上"，一条在 "左下"，钦定 $(i_1, j_1)$ 所在路线在 "右上"，因此枚举 $i_1(1<=i_1<=n)$， $j_1(1<=j_1<=m)$，$i_2(i_1<i_2<=n)$，$j_2(1<=j_2<j_1)$，这样保证 $(i_2, j_2)$ 所在线路与 $(i_1, j_1)$ 线路总不相交，转移时直接由上下左右移动一个位置转移即可，具体的，$dp[i_1][j_1][i_2][j_2] = max(dp[i_1-1][j_1][i_2-1][j_2], dp[i_1][j_1-1][i_2-1][j_2], dp[i_1-1][j_1][i_2][j_2-1], dp[i_1][j_1-1][i_2][j_2-1])+val[i_1][j_1]+val[i_2][j_2]$，其中 $val[i][j]$ 表示在 $(i, j)$ 位置的同学的好心程度

不需要另设边界条件，因为没有开始传时最大价值就是 $0$；注意最终答案为 $dp[n-1][m][n][m-1]$，因为显然 "右上" 线的终点为 $(n-1, m)$ 即右下角上方，"左上" 线的终点为 $(n, m-1)$ 即右下角左方

时间复杂度 $O(N^2M^2)$

代码中的 $n$ 与 $m$ 和思路中是反过来的

```cpp
#include<bits/stdc++.h>
using namespace std;
const int MAXM = 55, MAXN = 55;

int m, n;
int mp[MAXM][MAXN];
int f[MAXM][MAXN][MAXM][MAXN];

int main(){
	scanf("%d%d", &m, &n);
	
	for (int i=1; i<=m; i++){
		for (int j=1; j<=n; j++){
			scanf("%d", &mp[i][j]);
		}
	}
	
	for (int i=1; i<=m; i++){
		for (int j=1; j<=n; j++){
			for (int k=i+1; k<=m; k++){
				for (int l=1; l<j; l++){
					f[i][j][k][l] = max(
						max(f[i][j-1][k][l-1], f[i][j-1][k-1][l]), 
						max(f[i-1][j][k][l-1], f[i-1][j][k-1][l])
					);
					f[i][j][k][l] += (mp[i][j]+mp[k][l]);
				}
			}
		}
	}
	
	printf("%d", f[m-1][n][m][n-1]);
	return 0;
}
```