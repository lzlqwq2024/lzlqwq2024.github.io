---
date:
  created: 2024-04-17
  updated: 2024-04-17
draft: false
pin: false
comments: true

categories:
  - 字符串
tags:
  - 学习记录
---

# KMP

## 1. 前缀函数

### 1.1 定义
给定一长度为 $n$ 的字符串 $s$，定义其前缀函数为一长度为 $n$ 的数组 $nex$，其中的每一位 $nex[i]$，表示子串 $s[0...i]$ 的最长公共前后缀的长度；特别的，$nex[0]=0$

例如对于字符串 $aabaab$，其前缀函数 $nex[0]=0, nex[1]=1, nex[2]=0, nex[3]=1, nex[4]=2, nex[5]=3$ 

### 1.2 计算方法
假定对于长度为 $n$ 的字符串 $s$, 已知 $nex[i]$，要求 $nex[i+1]$

1. 若 $s[nex[i]]=s[i+1]$，则 $nex[i+1]=nex[i]+1$
![s[nex[i]]=s[i+1]](https://cdn.luogu.com.cn/upload/image_hosting/yk9kj34k.png)
2. 若 $s[nex[i]]≠s[i+1]$，那么我们希望对于子串 $s[0...i]$，找到仅次于 $nex[i]$ 的第二长度 $j$，使得 $s[0...j-1]=s[i-j+1...i]$
![s[nex[i]]≠s[i+1]](https://cdn.luogu.com.cn/upload/image_hosting/vgjc0mcf.png)

    因此，我们只需再次比较 $s[j]$ 与 $s[i+1]$，若相等则有 $nex[i+1]=j+1$，于是问题便转化为在子串 $s[0...nex[i]-1]$ 中找到最长的公共前后缀长度 $j$，这与原问题相同，即 $j=nex[nex[i]-1]$。同理，次于 $j$ 的第二长度 $j^{(2)}$ 满足  $j^{(2)}=nex[j-1]$
    
    最终可以得到一个关于 $j$ 的递推式：$j^{(n)}=nex[j^{(n-1)}-1](j^{(n-1)}>0)$
    
### 1.3 算法
```cpp
void Getnex(string s, int *nex){
	nex[0] = 0;
	
	for (int i=1; i<s.size(); i++){
		int j = nex[i-1];
		
		while (j>0 && s[i]!=s[j]){
			j = nex[j-1];
		}
		
		if (s[i] == s[j]){
			j++;
		}
		nex[i] = j;
	}
}
```

## 2. KMP 算法

### 2.1 面向问题
给定两字符串 $t$ 和 $s$，找到并输出 $s$ 在 $t$ 中的所有出现

### 2.2 过程
设字符串 $s$ 的长度为 $n$，字符串 $t$ 的长度为 $m$

构造一个字符串 $t'=s+\#+t$，其中 $\#$ 为一个在字符串 $s$ 与 $t$ 中均不出现的分隔符，求得此字符串的前缀函数 $nex$

对于 $nex[i](i≥n+1)$，若 $nex[i]=n$，则说明 $s$ 完整的出现在该位置 (右端点位于 $nex[i]$)，但 $i$ 是对于字符串 $s'$ 的位置，因此在减去 $s+\#$ 的长度后，易得知字符串 $s$ 在 $t$ 的 $i-(n-1)-(n+1)=i-2n$ 处出现

### 2.3 例题
例题：[P3375 [模版] KMP](https://www.luogu.com.cn/problem/P3375)

时间复杂度为 $O(n+m)$，空间复杂度为 $O(n)$

```cpp
#include<bits/stdc++.h>
using namespace std;
const int MAXN = 1e7+5;

string s1, s2;
int nex[MAXN], nexans[MAXN];

void Getnex(string s, int *nex){
	nex[0] = 0;
	
	for (int i=1; i<s.size(); i++){
		int j = nex[i-1];
		
		while (j>0 && s[i]!=s[j]){
			j = nex[j-1];
		}
		
		if (s[i] == s[j]){
			j++;
		}
		nex[i] = j;
	}
}

void work(string s1, string s2){    // search s1 in s2
	string s = s1+'#'+s2;
	Getnex(s1+'#'+s2, nex);
	
	for (int i=s1.size()+1; i<=s1.size()+s2.size(); i++){
		if (nex[i] == s1.size()){
			printf("%d\n", i-2*s1.size()+1);
		}
	}	
}

int main(){
	cin >> s1 >> s2;
	
	work(s2, s1);
	Getnex(s2, nexans);
	
	for (int i=0; i<s2.size(); i++){
		printf("%d ", nexans[i]);	
	}
	return 0;
}
```

## 3. KMP 应用

### 3.1 字符串的周期
对长度为 $n$ 的字符串 $s$ 以及 $0＜p≤n$，若 $s[i]=s[i+p]$ 对所有 $i∈[0, n-p-1]$ 成立，则称 $p$ 是 $s$ 的周期，即将 $s$ 的子串 $s[0...p-1]$ 重复无限次得到字符串 $s'$，有 $s$ 是 $s'$ 的子串 (如 $abcdab$ 为 $abcd$ 重复无限次得到的字符串的子串，则 $4$ 为 $abcdab$ 的周期)

对长度为 $n$ 的字符串 $s$，若 $s$ 有长度为 $0≤r＜n$ 的公共前后缀，则称 $s$ 长度为 $r$ 的公共前后缀为 $s$ 的 $border$

1. 对长度为 $n$ 的字符串 $s$，若其有一个长度为 $r$ 的 $border$，可以发现 $n-r$ 是 $s$ 的周期，最短的周期为 $n-nex[n-1]$

2. 特别的，对长度为 $n$ 的字符串 $s$ 以及长度为 $p$ 的前缀 $s'$，若 $s$ 可以通过 $s'$ 重复 $k$ 次得到，称 $s'$ 为周期性的，其周期为 $k$

    对于长度为 $r$ 的 $border$，当且仅当 $n-r \equiv 0 (\bmod n)$ 时 (即 $n-r$ 整除 $n$)，该 $border$ 为周期性的，其周期为 $\frac{n}{n-r}$