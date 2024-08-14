---
date:
  created: 2024-07-20
  updated: 2024-07-20
draft: false
pin: false
comments: true

categories:
  - 做题记录
tags:
  - 树形DP基础
---

## P2015

[P2015](https://www.luogu.com.cn/problem/P2015)

典型的树上背包模型。我们设 $dp[i][j]$ 表示在以 $i$ 为根的子树中保留 $j$ 条边的最大价值；按照树形 $DP$ 的一般套路，在节点 $node$ 处，首先递归搜索儿子的子树，再进行转移；

转移时，设 $j$ 为当前以 $node$ 为根的子树选择的边数，$k$ 为以 $child$ 为根的子树选择的边数，连接节点 $(i, j)$ 的边的权值为 $val[i][j]$，转移方程即为 $dp[node][j] = \sum_{child \in son(node)}max(dp[node][j-k-1]+dp[child][k]+val[node][child])$，注意这里因为 **还有 $(node, child)$ 的一条边**，所以转移方程中第二维 $j-k-1$ 与 $k$ 的和是 $j-1$ 而非 $j$

实现时，我们可以考虑记录节点 $i$ 子树中边的个数 $siz[i]$，方便判断边界；$siz[i]$ 的转移方程为 $siz[node] = \sum_{child \in son(node)}siz[child]+1$，同样别忘了考虑 $(node, child)$ 的一条边

答案即为 $dp[1][Q]$

代码

```cpp
#include<bits/stdc++.h>
using namespace std;
const int MAXN = 105, MAXQ = 105;

struct Edge{
	int to;
	int weig;
};

int N, Q;
vector <Edge> G[MAXN];
int dp[MAXN][MAXQ], siz[MAXN];

void dfs(int node, int fa){
	for (int i=0; i<G[node].size(); i++){
		if (G[node][i].to == fa) continue;
		dfs(G[node][i].to, node);
		siz[node] += siz[G[node][i].to]+1;
		for (int j=min(siz[node], Q); j>=1; j--){
			for (int k=0; k<=min(siz[G[node][i].to], j-1); k++){
				dp[node][j] = max(dp[node][j], dp[node][j-k-1]+dp[G[node][i].to][k]+G[node][i].weig);
			}
		}
	}
}

int main(){
	scanf("%d%d", &N, &Q);
	
	for (int i=1; i<=N-1; i++){
		int node1, node2, weig;
		scanf("%d%d%d", &node1, &node2, &weig);
		G[node1].push_back({node2, weig});
		G[node2].push_back({node1, weig});
	}
	
	dfs(1, 1);
	printf("%d", dp[1][Q]);
	return 0;
}
```

## P2014

[P2014](https://www.luogu.com.cn/problem/P2014)

仍然是树上背包问题。转移方程与 $P2015$ 是类似的；实现时，有几个细节略有不同；我们将每门课的先修课向该课连边，最终会形成一个森林，如果我们依次 $dp$ 每棵树并统计不太方便，因此考虑 **增加一个 $0$ 号点，作为所有没有先修课的课程的先修课**，设其价值为 $0$，再进行树形 $DP$ 即可；别忘了我们在这里增加了一个点，对枚举的上界与最终的答案选择会产生影响

```cpp
#include<bits/stdc++.h>
using namespace std;
const int MAXN = 105;

int N, M;
vector <int> G[MAXN];
int dp[MAXN][MAXN], siz[MAXN], in[MAXN], w[MAXN];

void dfs(int node, int fa){
	for (int i=0; i<G[node].size(); i++){
		if (G[node][i] == fa) continue;
		dfs(G[node][i], node);
		siz[node] += siz[G[node][i]]+1;
		for (int j=min(M+1, siz[node]); j>=1; j--){
			for (int k=1; k<=min(siz[G[node][i]], j-1); k++){
				dp[node][j] = max(dp[node][j], dp[node][j-k]+dp[G[node][i]][k]);
			}
		}
	}
}

int main(){
	scanf("%d%d", &N, &M);
	
	w[0] = 0;
	for (int i=1; i<=N; i++){
		int node, weig;
		scanf("%d%d", &node, &weig);
		G[node].push_back(i);
		in[i]++;
		w[i] = weig;
	}
	
	for (int i=1; i<=N; i++){
		if (in[i] == 1) siz[i]++;
		dp[i][1] = w[i];
	}
	
	dfs(0, 0);
	printf("%d", dp[0][M+1]);
	return 0;
}
```

## BNDSOJ DP-5-3 祖孙询问

[DP-5-3 祖孙询问](https://onlinejudge.bnds.cn/contest/problem?id=762&pid=2)

可以使用 $LCA$ 卡过，如果 $x$ 是 $y$ 的祖先那么 $lca(x, y) = x$，如果 $y$ 是 $x$ 的祖先那么 $lca(x, y) = y$，如果都不是那么 $lca(x, y)$ 既不是 $x$ 也不是 $y$；

但是，这道题中只需要判断 $x$ 与 $y$ 是否在同一子树中，没有必要使用 $LCA$；我们考虑与 $Tarjan$ 类似的思想，在对树进行 $dfs$ 时，我们开两个数组 $in[i], out[i]$，分别表示节点 $i$ 什么时候第一次被遍历，$i$ 什么时候退出遍历；

对于节点 $i$ 与 $j$，如果 $i$ 是 $j$ 的祖先，那么 **一定有** $in[i] < in[j]\  \&\&\  out[i] \ge out[j]$，证明是显然的；如果两节点不在一个子树中，那么两节点的 $in, out$ 区间不交

代码

```cpp
#include<bits/stdc++.h>
using namespace std;
const int MAXN = 1e5+5;

int n, m, s, diff=0;
int in[MAXN], out[MAXN];
vector <int> G[MAXN];

void dfs(int node, int fa){
	diff++;
	in[node] = diff;
	for (int i=0; i<G[node].size(); i++){
		if (G[node][i] == fa) continue;
		dfs(G[node][i], node);
	}
	out[node] = diff;
}

int main(){
	scanf("%d", &n);
	
	for (int i=1; i<=n; i++){
		int node1, node2;
		scanf("%d%d", &node1, &node2);
		if (node2 == -1){
			s = node1; 
			continue;
		}
		G[node1].push_back(node2);
		G[node2].push_back(node1);
	}
	
	dfs(s, s);
	
	scanf("%d", &m);
	
	for (int i=1; i<=m; i++){
		int node1, node2;
		scanf("%d%d", &node1, &node2);
		if (in[node1] < in[node2] && out[node1] >= out[node2]) printf("1\n");
		else if (in[node1] > in[node2] && out[node1] <= out[node2]) printf("2\n");
		else printf("0\n");
	}
	return 0;
}
```

## BNDSOJ DP-5-5 容易题

[DP-5-5 容易题](https://onlinejudge.bnds.cn/contest/problem?id=762&pid=4#problem-anchor)

由于我们从一个点开始 $dfs$，达到的最远点必定是直径的一个端点，因此先求出直径 $\delta(s, t)$，再分别以 $s$ 与 $t$ 为根进行两次 $dfs$ 求出其他点到这两点的距离即可；对于点 $node$，答案即为 $max\{\delta(node, s), \delta(node, t)\}$

代码

```cpp
#include<bits/stdc++.h>
using namespace std;
const int MAXN = 10005;

int N, c, node1, node2;
int d[MAXN], d1[MAXN], d2[MAXN];
vector <int> G[MAXN];

void dfs(int node, int fa){
	for (int i=0; i<G[node].size(); i++){
		if (G[node][i] == fa) continue;
		d[G[node][i]] = d[node]+1;
		if (d[G[node][i]] > d[c]) c = G[node][i];
		dfs(G[node][i], node);
	}	
}

void get1(int node, int fa){
	for (int i=0; i<G[node].size(); i++){
		if (G[node][i] == fa) continue;
		d1[G[node][i]] = d1[node]+1;
		get1(G[node][i], node);
	}
}
void get2(int node, int fa){
	for (int i=0; i<G[node].size(); i++){
		if (G[node][i] == fa) continue;
		d2[G[node][i]] = d2[node]+1;
		get2(G[node][i], node);
	}
}

int main(){
	scanf("%d", &N);
	
	for (int i=1; i<=N-1; i++){
		int node1, node2;
		scanf("%d%d", &node1, &node2);
		G[node1].push_back(node2);
		G[node2].push_back(node1);
	}
	
	dfs(1, 1);
	d[c] = 0;
	node1 = c;
	dfs(c, c);
	node2 = c;
	
	get1(node1, node1);
	get2(node2, node2);
	
	for (int i=1; i<=N; i++){
		printf("%d\n", max(d1[i], d2[i]));
	}
	return 0;
}
```

## P2016

[P2016](https://www.luogu.com.cn/problem/P2016)

题意即为求给定树的最小点覆盖；我们注意到，一条边的两个端点至少要选一个，因此设 $dp[i][0/1]$ 表示在点 $i$ 处设置 / 不设置士兵所花的最小代价；

转移方程即为 $dp[i][1] = \sum_{v \in son(i)}min(dp[v][0], dp[v][1])$，$dp[i][0] = \sum_{v \in son(i)}dp[v][1]$，在 $dfs$ 转移前别忘了设初值 $dp[i][1] = 1$

代码

```cpp
#include<bits/stdc++.h>
using namespace std;
const int MAXN = 2e3+5, INF = 2e9;

int n, id, k, node;
vector <int> G[MAXN];
int dp[MAXN][2];

void dfs(int node, int fa){
	dp[node][1] = 1;
	for (int i=0; i<G[node].size(); i++){
		if (G[node][i] == fa) continue;
		dfs(G[node][i], node);
		dp[node][1] += min(dp[G[node][i]][0], dp[G[node][i]][1]);
		dp[node][0] += dp[G[node][i]][1];
	}
}

int main(){
	scanf("%d", &n);
	
	for (int i=1; i<=n; i++){
		scanf("%d%d", &id, &k);
		for (int i=1; i<=k; i++){
			scanf("%d", &node);
			G[id].push_back(node);
			G[node].push_back(id);
		}
	}

	dfs(0, 0);
	printf("%d", min(dp[0][0], dp[0][1]));
	return 0;
}
```

## P2458

[P2458](https://www.luogu.com.cn/problem/P2458)

与上一道题略有不同，上一道题希望覆盖所有的边，这道题希望覆盖所有的点；由于一个点被覆盖可能通过三种方式，我们考虑设 $dp[i][0/1/2]$，表示在该点处设置保安 / 在该点的 **父亲** 处设置保安 / 在该点的 **儿子** 处设置保安的最小代价

我们一个一个来看，对于 $dp[i][0]$，由于当前节点已经被覆盖，儿子节点 **可以选择任意方式**，因此转移方程为 $dp[i][0] = \sum_{v \in son(i)}min(dp[v][0], dp[v][1], dp[v][2])$

对于 $dp[i][1]$，设此时 $i$ 的孩子为 $v$，由于 $i$ 现在并没有设置保安，因此 **不能通过 $dp[v][1]$ 转移而来**；因此把这种情况挖掉，转移方程即为 $dp[i][1] = \sum_{v \in son(i)}min(dp[i][0], dp[i][2])$

对于 $dp[i][2]$，同理此时不能通过 $dp[v][1]$ 转移而来，因此转移方程即为 $dp[i][2] = \sum_{v \in son(i)}min(dp[i][0], dp[i][2])$ ……了吗？事情没有这么简单；我们发现，如果节点 $i$ 的 **所有儿子 $v$ 都选择通过 $v$ 的儿子来覆盖** (即通过 $dp[v][2]$ 转移而来)，那么实际上所有的儿子 $v$ 上都没有保安，这与状态设计不符；这种情况下，我们只能考虑选取一个儿子 $v$，将由 $dp[v][2]$ 转移换成由 $dp[v][0]$ 转移，因此我们也要在转移时 **记录每个儿子 $dp[v][0]-dp[v][2]$ 的最小值 $extra$，最终特判如果全都选择由 $dp[v][2]$ 转移而来，则将结果加上 $extra$**

实现时，别忘了将叶子节点的 $dp[i][2]$ 设为 $+\infty$，同时将每个 $dp[i][0]$ 赋初值 $v[i]$

代码

```cpp
#include<bits/stdc++.h>
using namespace std;
const int MAXN = 2005, INF = 1e9;

int n, start, id, k, m, child, cover;
int r[MAXN];
bool nroot[MAXN], leaf[MAXN];

vector<int>G[MAXN];
long long dp[MAXN][3];

void dfs(int node){
	dp[node][0] = r[node];
	bool flag = true;
	long long cover = 1ll*INF;
	
	for (int i=0; i<G[node].size(); i++){
		dfs(G[node][i]);
		
		dp[node][0] += min(dp[G[node][i]][0], min(dp[G[node][i]][1], dp[G[node][i]][2]));
		dp[node][1] += min(dp[G[node][i]][0], dp[G[node][i]][2]);
		
		if (dp[G[node][i]][0] < dp[G[node][i]][2]){
			flag = false;
			dp[node][2] += dp[G[node][i]][0];
		}
		else{
			cover = min(1ll*cover, dp[G[node][i]][0]-dp[G[node][i]][2]);
			dp[node][2] += dp[G[node][i]][2];
		}
	}
	
	if (flag == true){
		dp[node][2] += cover;
	}
}

int main(){
	scanf("%d", &n);
	
	for (int i=1; i<=n; i++){
		scanf("%d%d%d", &id, &k, &m);
		r[id] = k;
		if (m == 0) leaf[id] = true;
		for (int i=1; i<=m; i++){
			scanf("%d", &child);
			nroot[child] = true;
			G[id].push_back(child);
		}
	}
	
	for (int i=1; i<=n; i++){
		if (leaf[i] == true){
			dp[i][2] = INF;	
		}
		if (nroot[i] == false){
			start = i;
		}
	}
	
	dfs(start);
	printf("%lld", min(dp[start][0], dp[start][2]));
	return 0;
}
```



## BNDSOJ DP-5-8 旅游规划

[DP-5-8 旅游规划](https://onlinejudge.bnds.cn/contest/problem?id=762&pid=7)

题意为让我们求出所有在直径上的节点 (直径可能有很多条)；显然只按常规操作进行二次 $dfs$ 无法解决问题 (因为只能求出一条直径上的节点)，但又因为暴力选取每个点分别进行 $dfs$ 时间复杂度为 $O(n^2)$，会超时，我们希望优化算法

首先，注意到任选树上一节点 $s$ 进行 $dfs$，到达的节点肯定是直径的端点，这很显然，反证即可；我们在 $dfs$ 中维护以 $s$ 为根时所有节点的深度 $d[i]$ 与最终直径的长度 $maxd$，搜索完成后枚举每个节点 $i$，记录下所有 $d[i] = maxd$ 的节点 $i$；由于我们可以任意选取直径，易知所有的节点 $i$ 都为某一条直径的端点，同时，**所有的直径都至少有一个端点在这些节点中**

接下来，如果对于每个端点都再进行一次 $dfs$，仍然会超时；我们考虑只选取一个端点 $node$ 再进行 $dfs$，按照同样的方法记录下所有离 $node$ 最远的节点 $k$；同理，所有的节点 $k$ 也都为某一条直径的端点；又因为 **所有直径中点都相交**，则 **所有的节点 $k$ 和所有的节点 $i$ 已经覆盖了所有的直径的端点**；

现在我们已经得到了所有的直径的端点，但题目让我们求所有直径上的点，怎么办呢？我们 **在第二次 $dfs$ 中** 考虑记录每个节点的父亲，依次往上跳即可；那为什么可以覆盖所有直径上的点？我们设一条直径为 $\delta(s, t)$，另一条为 $\delta(p, q)$，现在我们将节点 $s$ 作为根跑 $dfs$，由上，易知 $\delta(s, t)$ 与 $\delta(p, q)$ 有交点，设为 $mid$，在 $\delta(p, s)$ 上的所有节点，都能够在 $p \rightarrow mid \rightarrow s$ 这么跳时被覆盖，同理在 $\delta(q, s)$ 上的所有节点都能够在 $q \rightarrow mid \rightarrow s$ 这么跳时被覆盖；因此证毕

代码

```cpp
#include<bits/stdc++.h>
using namespace std;
const int MAXN = 2e5+5;

int n, maxd, nod;
int d[MAXN], jump[MAXN];
vector <int> G[MAXN];
set <int> res, ans;

void dfs(int node, int fa){
	for (int i=0; i<G[node].size(); i++){
		if (G[node][i] == fa) continue;
		d[G[node][i]] = d[node]+1;
		if (d[G[node][i]] > maxd) maxd = d[G[node][i]];
		dfs(G[node][i], node);
	}
}
void dfs2(int node, int fa){
	jump[node] = fa;
	for (int i=0; i<G[node].size(); i++){
		if (G[node][i] == fa) continue;
		d[G[node][i]] = d[node]+1;
		if (d[G[node][i]] > maxd) maxd = d[G[node][i]];
		dfs2(G[node][i], node);
	}
}

int main(){
	scanf("%d", &n);
	
	for (int i=1; i<=n-1; i++){
		int node1, node2;
		scanf("%d%d", &node1, &node2);
		G[node1].push_back(node2);
		G[node2].push_back(node1);
	}
	
	dfs(0, -1);
	for (int i=1; i<=n; i++){
		if (d[i] == maxd){
			nod = i;
			res.insert(i);
		}
	}
	maxd = 0, d[nod] = 0;
	dfs2(nod, -1);
	for (int i=1; i<=n; i++){
		if (d[i] == maxd){
			res.insert(i);
		}
	}
	for (auto i=res.begin(); i!=res.end(); i++){
		int recent = (*i);
		while (recent != -1){
			ans.insert(recent);
			recent = jump[recent];
		}
	}
	for (auto i=ans.begin(); i!=ans.end(); i++){
		printf("%d\n", (*i));
	}
	return 0;
}
```