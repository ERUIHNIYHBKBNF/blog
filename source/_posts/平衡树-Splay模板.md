---
title: '[模板] 平衡树-Splay模板'
date: 2020-11-28 21:24:14
tags: qwq

---

 [先膜大佬%%%](https://www.cnblogs.com/zwfymqz/p/7896036.html)

持续**重构六次**代码之后终于过了板子题。。不容易\_(:з:broken_heart:」∠)\_



## 普通平衡树

板子： [P3369 【模板】普通平衡树](https://www.luogu.com.cn/problem/P3369)

实现功能：

1. 插入 *x* 数
2. 删除 *x* 数 (若有多个相同的数，因只删除一个)
3. 查询 *x* 数的排名 (排名定义为比当前数小的数的个数 +1+1 )
4. 查询排名为 *x* 的数
5. 求 *x* 的前驱 (前驱定义为小于 *x*，且最大的数)
6. 求 *x* 的后继 (后继定义为大于 *x*，且最小的数)

## 二叉搜索树 BST

一种特殊的二叉树， 保证每个节点与其左右儿子(若存在)的值存在关系 : $ val[lson] \lt val[np] \le val[rson]$ 。

于是就可以很容易维护 大小关系，排名，前驱后继什么的。

查询：`nxt = query_v < v[np] ? lson : rson;`

平均复杂度$O(\log N)$..然而这样在最坏情况下~~可能~~必然会被卡成一条链也就是复杂度 $O(N^2)$ (ﾉ ○ Д ○)ﾉ

于是就有了各种各种~~长度令人及其不适的~~平衡树。

## 伸展树 Splay

实现平衡的方法是 **尽可能把出现频率更高的点往上旋转** ，差不多就是每次查询的点都往上转什么的..还挺牛逼的~~（完全不懂为啥QAQ）~~

直接扔代码好了，按函数拆开qwq

### 准备工作

一些懒人用的define（别骂了别骂了..）和需要的变量：

```cpp
#include <cstdio>
#include <cstring>
#include <iostream>
#include <cstdlib>
#define rep(i, s, t) for (int i = s; i <= t; i++)
#define root e[0].ch[1] //人为规定0号点的右儿子为根
#define fa(x) e[x].fa
#define lson(x) e[x].ch[0]
#define rson(x) e[x].ch[1]
using namespace std;
const int MAXN = 1e5 + 233, INF = 0x7fffffff;
struct node
{
    int v, fa, ch[2], sum, rec; //sum为子树大小（含重复）， rec为当前点的值重复次数。
} e[MAXN];
int n; //n为节点总数，包含已删除的，其实就是为了获得编号qwq
void update(int x) //更新节点信息
{
    e[x].sum = e[lson(x)].sum + e[rson(x)].sum + e[x].rec;
}
bool ident(int x) //判断当前点是左儿子还是右儿子
{
    return lson(fa(x)) != x;
}
void connect(int x, int fa, bool son) //把 x 作为 son儿子 连接到 fa 上
{
    e[fa].ch[son] = x;
    e[x].fa = fa;
}
```

### Splay核心操作

#### 旋转

旋转 X 点就是要把 X 的深度减少一层。简单说就是把他跟他爹互换位置emmm

旋转操作如图：

<img src="https://s3.ax1x.com/2020/12/01/Dh9Swn.jpg" width="500px">

<p align="center">好像是经典老图了qwq</p>

正向是旋转X，反向是旋转Y。不难看出无论怎样旋转，都存在如下规律(以旋转 X 为例)：

* 记 X 是 Y 的 yson 儿子(0表示左, 1表示右)，由BST的性质，旋转后 Y 变成了 X 的 yson ^ 1 儿子。
* 原本 X 的 yson ^ 1 儿子记为 B ，由BST的性质， 旋转后 B 变成了 Y 的 yson 儿子。

Code:

```cpp
void rotate(int x)
{
    int y = fa(x), r = fa(y);
    bool yson = ident(x), Rson = ident(y); //小心rson撞名字qwq
    int b = e[x].ch[yson ^ 1];
    connect(b, y, yson);
    connect(x, r, Rson);
    connect(y, x, yson ^ 1);
    update(y); //记得从下往上更新状态
    update(x);
}
```

#### Splay

对于查询频率更高的点， 我们要把它尽可能向上旋转（即每次旋转到根），然而出题人怎么会放过如此傻乎乎的操作，，，砰！你炸了\~\~\~

> 对于单调数据，树会退化成一条链，然后每次move最深的点就被卡掉啦~

比如下图， 查询 4 的时候这样旋转， 如果接下来查询3， 2， 1...

<img src="https://s3.ax1x.com/2020/12/01/Dh9VOJ.jpg" width="750px">

所以改用双旋法，，*可以用势能法证明splay是均摊*$ O(log N)$*的*， ~~然而咱不会证。~~

具体怎么做，就是当 X 和 fa(X) 在一条直线上的时候，先旋转 fa(X) 再旋转 X，~~然而在咱也不知道为啥。~~

<img src="https://s3.ax1x.com/2020/12/01/Dh91SO.jpg" width="750px">

Code:

```cpp
void splay(int x, int to) //把 x 旋转到 to 位置
{
    to = fa(to); //用fa(to)作为标识，这样好写。毕竟最后一步，原to与x的父子关系会改变，不太好判断。
    while (fa(x) != to)
    {
        int y = fa(x);
        if (fa(y) == to)
            rotate(x);
        else if (ident(x) == ident(y)) //x与fa(x)在一条直线上
            rotate(y), rotate(x);
        else
            rotate(x), rotate(x);
    }
}
```

### 其它操作

二叉搜索树的相关操作，不要忘记Slpay就行qwq

#### 查询数字 x 的位置

```cpp
int find(int v)
{
    int np = root; //从根开始查找
    while (1)
    {
        if (e[np].v == v) //找到，返回
        {
            splay(np, root);//别忘记Splay,**注意查询点已经被旋转到root**
            return np;
        }
        bool son = v > e[np].v; //根据BST的性质继续查找
        if (e[np].ch[son])
            np = e[np].ch[son];
        else //查找不到
            return np; //这里return np便于配合下文rnk函数使用
    }
}
```

#### 插入数字 x

```cpp
void creat(int v, int fa) //创建一个新节点，值是 v，节点父亲为 fa
{
    e[++n].fa = fa;
    e[n].v = v;
    e[n].rec = e[n].sum = 1;
}
void insert(int v) //插入数值 v
{
    if (!root) //当前树为空
    {
        creat(v, 0);
        root = n;
        return;
    }
    int np = root; //查找该值是否已经存在
    while (1)
    {
        e[np].sum++; //肯定插在当前点的子树上，总数+1
        if (e[np].v == v) //该值存在
        {
            e[np].rec++;
            splay(np, root);
            return;
        }
        bool son = v > e[np].v;
        if (!e[np].ch[son]) //不存在则创建新的节点
        {
            creat(v, np);
            e[np].ch[son] = n;
            splay(n, root);
            return;
        }
        np = e[np].ch[son];
    }
}
```

#### 删除数字 x

```cpp
void del(int v)
{
    int np = find(v); //题目好像保证 v 一定在树里，不然find就爆炸了qwq
    if (!np)
        return;
    if (e[np].rec > 1) //重复次数多于1，直接-1
    {
        e[np].rec--;
        e[np].sum--;
        return;
    }
    //满足BST性质的条件下删除节点
    //**注意np已经被旋转到了root**
    //case1:无儿子，直接删除节点
    if (!lson(np) && !rson(np))
    {
        root = 0;
        return;
    }
    //case2:无左儿子,直接用右儿子顶替
    if (!lson(np))
    {
        root = rson(np);
        fa(root) = 0;
        return;
    }
    //case3:查询左子树的最大值（即在左儿子开始，一直往右走），并用其顶替当前点
    int lmax = lson(np);
    while (rson(lmax))
        lmax = rson(lmax);
    splay(lmax, lson(np)); //旋转至要删除的点这里
    connect(rson(np), lmax, 1);
    connect(lmax, 0, 1);
    update(lmax); //容易写漏qwq
}
```

#### 查询数字 x 排名

```cpp
int rnk(int v) //因为在find时已经被旋转至根，所以直接这样写qwq
{
    return e[lson(find(v))].sum + 1;
}
//题目数据好像保证了查询数字一定在树中？
```

#### 查询排名为 x 的数

```cpp
int kth(int k) //写法好像有很多，这个是自己yy的qwq
{
	int np = root, nums = 0; //nums是小于当前数的数字个数
	while (1)
	{
		if (nums + e[lson(np)].sum < k && k <= nums + e[lson(np)].sum + e[np].rec)
			break; //查询到
		if (k <= nums + e[lson(np)].sum) //继续查询
			np = lson(np);
		else //由BST性质,往右走的话要使nums加上左子树的大小
		{
			nums += e[lson(np)].sum + e[np].rec;
			np = e[np].ch[1];
		}
	}
	splay(np, root); //别忘记Splay
	return e[np].v;
}
```

#### 查询 x 的前驱 / 后继

```cpp
int lower(int v)
{
    int np = root, ans = -INF;
    while (np) //根据BST性质，一定是查到底为止qwq
    {
        if (v > e[np].v) //可能更新的ans
            ans = max(ans, e[np].v);
        bool son = v > e[np].v;
        np = e[np].ch[son];
    }
    return ans;
}
int upper(int v) //对称写emmm
{
    int np = root, ans = INF;
    while (np)
    {
        if (e[np].v > v)
            ans = min(ans, e[np].v);
        bool son = v >= e[np].v;
        np = e[np].ch[son];
    }
    return ans;
}
```

#### 顺带一个主函数 ~~（方便粘板子）~~

```cpp
int main()
{
//	freopen("in.txt", "r", stdin);
	int N;
	scanf("%d", &N);
	int opt, x;
	while (N--)
	{
		scanf("%d%d", &opt, &x);
		if (opt == 1)
			insert(x);
		else if (opt == 2)
			del(x);
        else if (opt == 3)
			printf("%d\n", rnk(x));
		else if (opt == 4)
			printf("%d\n", kth(x));
		else if (opt == 5)
			printf("%d\n", lower(x));
		else
			printf("%d\n", upper(x));
	}
	return 0;
}
```

先写这些好了qwq

~~然而这也会咕咕咕到寒假~~

