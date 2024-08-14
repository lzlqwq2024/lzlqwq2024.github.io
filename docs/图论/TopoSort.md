---
date:
  created: 2024-04-17
  updated: 2024-04-17
draft: false
pin: false
comments: true

categories:
  - 图论
tags:
  - 学习记录
---

# TopoSort

## 1. 拓扑排序

### 1.1 面向问题
在一个 $DAG$ (有向无环图) 中对所有节点以线性方式进行排序，使得对于由任意一条有向边 $(u, v)$ 相连的节点 $u$, $v$，都有 $u$ 在 $v$ 的前面

宏观意义上，拓扑排序可以使得排在前面的节点不依赖于排在后面的节点

### 1.2 构造拓扑序列步骤
1. 从图中选择一个入度为 $0$ 的节点
2. 输出该顶点，从图中删去此节点以及此节点所有的出边

重复上面两步，直到所有定点都被输出，拓扑排序完成，或者图中不存在入度为 $0$ 的节点 (此时说明图是有环图)

### 1.3 例题
例题：[B3644 [模板] 拓扑排序](https://www.luogu.com.cn/problem/B3644)

对于图 $G = (V, E)$，时间复杂度为 $O(E+V)$

```cpp
#include <bits/stdc++.h>
using namespace std;
const int MAXN = 505;

int N;
int in[MAXN];
vector <int> G[MAXN];

void TopoSort(){
	vector <int> L;
	queue <int> S;
	
	for (int i=1; i<=N; i++){
		if (in[i] == 0){
			S.push(i);
		}
	}
	
	while (!S.empty()){
		int recent = S.front();
		S.pop();
		L.push_back(recent);
		
		for (int i=0; i<G[recent].size(); i++){
			in[G[recent][i]]--;
			if (in[G[recent][i]] == 0){
				S.push(G[recent][i]);
			}
		}
	}
	
	for (int i=0; i<L.size(); i++){
		printf("%d ", L[i]);
	}
}

int main() {
	scanf("%d", &N);
	
	for (int i=1; i<=N; i++){
		while (true){
			int recent;
			scanf("%d", &recent);
			if (recent == 0){
				break;
			}
			else{
				G[i].push_back(recent);
				in[recent]++;
			}
		}
	}
	
	TopoSort();
	return 0;
}
```

## 2. AOE 网与关键路径

### 2.1 定义
$AOE$ 网是一个带权有向无环图，其中，顶点表示事件，边表示事件持续的时间，$AOE$ 网应该是无环的，且存在唯一入度为 $0$ 的起始节点 (源点)，以及唯一出度为 $0$ 的完成顶点 (汇点)

$AOE$ 网中的有些活动是可以并行进行的，所以完成整个工程的最短时间是从源点到汇点的最长活动路径长度；最长的活动路径也是关键路径，决定了工程完成的总时间

### 2.2 概念
- 活动：$AOE$ 网中，边表示活动，边的权值表示活动持续的时间，活动在事件被触发后开始
- 事件：$AOE$ 网中，节点表示事件，事件可以被触发
- 边 (活动) $a_{j}$ 的最早开始时间：源点到该边起点的最长路径长度，记为 $e(j)$
- 边 (活动) $a_{j}$ 的最迟开始时间：在不推迟整个工期的前提下，工程达到边起点所表示的状态最晚能容忍的时间
- 节点 (事件) $v_{j}$ 的最早发生时间：源点到该节点的最长路径长度，记为 $ve(j)$，显然 $ve(j)=e(j)$
- 节点 (事件) $v_{j}$ 的最迟发生时间：在不推迟整个工期的前提下，工程达到节点所表示的状态最晚能容忍的时间，记为 $vl(j)$
- 关键路径：$AOE$ 网中源点到汇点的最长路径的长度
- 关键活动：关键路径上的活动，最早开始时间和最迟开始时间相等

### 2.3 最早和最迟开始时间的递推关系
$ve(j) = max(ve(k) + dut(<k, j>))$

按拓扑顺序从前往后递推，节点的最早开始时间为结束了该顶点所有依赖的活动的时间 (活动可以并行)，而依赖的活动可能不止一个，因此 $ve(k) + dut(<k, j>) ≤ ve(j)$，因此联立取其最大值

$vl(j) = min(vl(k) - dut(<j, k>))$

按拓扑顺序从后往前递推，由于后继结点的最迟开始时间必然大于等于当前节点的最迟开始时间加上该边的权值，因此 $vl(k) - dut(<j, k>) ≥ vl(j)$，因此联立取其最小值

### 2.4 关键路径算法
1. 输入图，建立 $AOE$ 网
2. 从源点 $v_{0}$ 出发，令 $ve[0]=0$，按照拓扑排序求其余各个顶点的最早发生时间 $ve[i]$
3. 从汇点 $v_{n}$ 出发，令 $vl[n-1]=ve[n-1]$，按照逆拓扑顺序求其余各顶点的最迟发生时间 $vl[i]$
4. 对于任意点 $k$，若 $vl[k]=ve[k]$ 则该节点在关键路径上

### 2.5 例题
例题：[BNDSOJ - 拓扑排序 - 1.关键路径](https://onlinejudge.bnds.cn/contest/problem?id=629&pid=0#problem-anchor)

```cpp
#include<bits/stdc++.h>
using namespace std;
int n, weig;
int in[105], in_[105], ve[105], vl[105];

struct Edge{
	int to;
	int weig;
};

vector <Edge> G[105];
vector <Edge> G_[105];

void TopoSort(){
	vector <int> L;
	queue <int> S;
	
	for (int i=1; i<=n; i++){
		if (in[i] == 0){
			S.push(i);
		}	
	}
	
	while (!S.empty()){
		int recent = S.front();
		S.pop();
		L.push_back(recent);
		
		for (int i=0; i<G[recent].size(); i++){
			in[G[recent][i].to]--;
			ve[G[recent][i].to] = max(ve[G[recent][i].to], ve[recent]+G[recent][i].weig);
			if (in[G[recent][i].to] == 0){
				S.push(G[recent][i].to);
			}
		}
	}
	
	/*
	printf("ve\n");
	for (int i=0; i<L.size(); i++){
		printf("%d %d\n", L[i], ve[L[i]]);	
	}
	*/
}

void TopoSort_(){
	vector <int> L;
	queue <int> S;
	
	for (int i=1; i<=n; i++){
		if (in_[i] == 0){
			S.push(i);
		}	
	}
	
	while (!S.empty()){
		int recent = S.front();
		S.pop();
		L.push_back(recent);
		
		for (int i=0; i<G_[recent].size(); i++){
			in_[G_[recent][i].to]--;
			vl[G_[recent][i].to] = min(vl[G_[recent][i].to], vl[recent]-G_[recent][i].weig);
			if (in_[G_[recent][i].to] == 0){
				S.push(G_[recent][i].to);
			}
		}
	}
	
	/*
	printf("vl\n");
	for (int i=0; i<L.size(); i++){
		printf("%d %d\n", L[i], vl[L[i]]);	
	}
	*/
}

int main(){
	scanf("%d", &n);
	for (int i=1; i<=n; i++){
		for (int j=1; j<=n; j++){
			scanf("%d", &weig);
			if (weig != 0){
				G[i].push_back({j, weig});	
				in[j]++;
				
				G_[j].push_back({i, weig});
				in_[i]++;
			}	
		}
	}
	
	TopoSort();
	memset(vl, 63, sizeof(vl));
	vl[n] = ve[n];
	TopoSort_();
	
	//printf("ans\n");
	for (int i=1; i<=n; i++){
		if (ve[i] == vl[i]){
			printf("%d ", i);
		}
	}
	return 0;
}
```