---
title: '[杂题] 生成树 & LCA'
date: 2020-08-07 16:01:46
tags: acm, 模板
math: true

---

### Prim

半年前，计划里有那么一道题，直到退役都懒得去做.. [P1265 公路修建](https://www.luogu.com.cn/problem/P1265)

裸题写半个下午咱真是菜得不行了TAT（辣鸡dev又卡我。。懵逼了半小时，重启，解决问题（｀Δ´）！

##### 一句话口胡prim流程

​	先把随便一个点放入联通块，每次贪心找 离当前连通块最近的点 并将其加入连通块，再用该点更新 连通块外的点 与 当前连通块 的最小距离，把所有点加入为止。

这题就算prim的板子叭..

code:

```cpp
void prim()
{
	memset(dis, 0x7f, sizeof(dis)); //dis记录每个点到当前连通块的最小距离 
	dis[1] = 0; 
	for (int i = 1; i <= n; i++)
	{
		int minp = 0;  //找距离当前连通块最近的未连通的点 
		for (int j = 1; j <= n; j++) 
			if (!vis[j] && dis[j] < dis[minp])
				minp = j;
		vis[minp] = 1; //将该点加入连通块
		ans += dis[minp];
		for (int j = 1; j <= n; j++) //更新dis 
			if (!vis[j])
				dis[j] = min(dis[j], dist(pot[minp], pot[j]));
	}
}
```

### Kruskal + LCA

为什么这些经典老题咱都没做过qwq [P1967 货车运输](https://www.luogu.com.cn/discuss/lists?forumname=P1967)

嗯。。一定是平时太水了\_(:з」∠)\_

本题kruskal重构树(使两点间路径最小值最大)，再LCA（掉坑了好多次orz）。

##### 一句话口胡kruskal流程

​	把所有边按权值排序，用 _冰茶姬_ 判断是否有重复连接两个相同节点的边，加入 n - 1条边就是一棵树啦(๑＞ڡ＜)☆~~然而本题是森林~~

##### 一句话口胡倍增LCA

​	记录树上每个节点往上跳 2的整数幂次 的信息，然后都可以用这些表示..

code:（码风极丑警告

```cpp
#include <cstdio>
#include <iostream>
#include <algorithm>
#include <cstring>
using namespace std;

const int maxn = 1e4 + 233;
int n, m, q, tmp, anc[maxn], fr[maxn], dep[maxn], fa[maxn][20], minv[maxn][20]; 
struct edge1
{
	int u, v, w;
} e1[maxn << 3]; //e1是原图
struct edge2
{
	int to, ne, w;
} e2[maxn << 3]; //e2是重构的图
void add(int u,int v,int w)
{
	e2[++tmp].to = v;
	e2[tmp].w = w;
	e2[tmp].ne = fr[u];
	fr[u] = tmp;
}

//Kruskal
bool cmp(edge1 ed1, edge1 ed2)
{
	return ed1.w > ed2.w;
}
int getf(int x)
{
	return (x == anc[x]) ? x : anc[x] = getf(anc[x]);
}
void kruskal()
{
	sort(e1 + 1, e1 + m + 1, cmp);
	for (int i = 1; i <= n; i++) //anc记录并查集祖先
		anc[i] = i;
	int cnt = 0; //cnt记录已插入的边数
	for (int i = 1; i <= m; i++)
	{
		int xx = getf(e1[i].u), yy = getf(e1[i].v);
		if (xx != yy)
		{
			anc[yy] = xx; //别把原来的点接上咯。。
			add(e1[i].u, e1[i].v, e1[i].w);
			add(e1[i].v, e1[i].u, e1[i].w);
			if (++cnt == n - 1) //生成树即插入 n - 1 条边
				break;
		}
	}
}

//LCA
void dfs(int np, int fp, int w) //LCA初始化,np当前节点，fp父节点，w二者之间的边权
{
	dep[np] = dep[fp] + 1;
	fa[np][0] = fp; //fa[x][y]为x号节点往上跳 2 ^ y 步所到达的点
	minv[np][0] = w;//minv[x][y]为x号节点到fa[x][y]之间路径的最小权值
	for (int i = 0; i <=18; i++)
	{
		//跳 2 ^ (i + 1)步相当于先跳 2 ^ y 步再跳 2 ^ y 步
        fa[np][i + 1] = fa[fa[np][i]][i];
		minv[np][i + 1] = min(minv[np][i], minv[fa[np][i]][i]);
	}
	for (int k = fr[np]; k; k = e2[k].ne)
	{
		int tp = e2[k].to;
		if (tp != fp)
			dfs(tp, np, e2[k].w);
	}
}
int query(int x, int y)
{
	int ans = 0x7fffffff;
	if (getf(x) != getf(y)) //不在一棵树上直接return -1;
		return -1;
	if (dep[x] < dep[y]) //深的先跳，跳至同一深度再一起跳
		swap(x, y); 
	for(int  i = 19; i >= 0; i--)
	{
		if (dep[fa[x][i]] >= dep[y]) //还能再往上跳
		{
			ans = min(ans, minv[x][i]); //先维护ans再赋值节点。。。
			x = fa[x][i];
		}
	}
    if (x == y) return ans; //到同一点就直接返回
	for(int  i = 19; i >= 0; i--)
	{
		if (fa[x][i] != fa[y][i])
		{
            ans = min(ans, min(minv[x][i], minv[y][i])); //同上，卡了好久，嘤嘤嘤~
			x = fa[x][i];
			y = fa[y][i];
		}
	}
	ans = min(ans, min(minv[x][0], minv[y][0])); //最后还差 两点分别 到根的两条边
	return ans;
}

int main()
{
//	freopen("in.txt", "r", stdin);
	scanf("%d%d", &n, &m);
	for (int i = 1; i <= m; i++)
		scanf("%d%d%d", &e1[i].u, &e1[i].v, &e1[i].w);
	
	kruskal(); 
	
	scanf("%d", &q);
	for (int i = 1; i <= n; i++)
		if (!dep[i])
			dfs(i, 0, 0x7fffffff);
	int x, y;
	for (int i = 1; i <= q; i++)
	{
		scanf("%d%d", &x, &y);
		printf("%d\n", query(x, y));
	}
	
	return 0;
}
```

### Kruskal与Prim对比

* 同采用贪心思想，kruskal关注的是边，prim关注的是点。

* 对于稠密图，prim优势更大（想想kruskal对大量边排序就可怕qwq

* kruskal操作时可以保留边的信息，用于重构树。

### 次小生成树

​	emmm赛前没学，，现在把这个技能点出来叭，这里仍然用 Kruskal + Lca 做法。

##### 不严格次小生成树

​	不严格是因为权值总和可能相等。

算法流程：mx\[ i ][ j ]表示 最小生成树上两点 i 和 j 路径上的 最大边的权值，枚举所有非树边，将其替换 mx\[ i ][ j ] ，不断操作即可找出不严格次小生成树。

##### 严格次小生成树

​	满足次小生成树边权总和一定**大于**最小生成树边权边权总和。

算法流程：同上，记录路径上的 最大值 与 严格次大值 即可。

fake code:

```cpp
int m1 = 0, m2 = 0; //m1, m2分别是最大值，次大值
balabala...;
if (m1 < val) //val是当前边的权值
	return val - m1;
else if (!m2)
	return val - m2;
else
    return -1; //表示当前环上所有权值相等，插入当前边的方式无解。
```

